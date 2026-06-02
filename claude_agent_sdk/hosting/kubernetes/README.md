# Tier 3 — Kubernetes (pod-per-session)

> Part of the [Agent SDK hosting cookbook](../../07_Hosting_the_agent.ipynb).
> If you haven't picked a hosting tier yet, start there — it covers when a
> managed option is the better fit and when you actually need this.

Run the agent on a Kubernetes cluster where every session gets its own
isolated pod, with network-level controls ensuring agent pods can only reach
the Anthropic API.

```
                     ┌──────────────────────────────────────────────────┐
                     │                  Kubernetes                      │
                     │                                                  │
  curl / SDK ──────► Gateway (FastAPI)                                  │
                     │  ├─ creates/deletes agent pods via K8s API       │
                     │  ├─ routes /sessions/{id}/messages to right pod  │
                     │  └─ session → pod mapping stored in Redis        │
                     │                                                  │
              ┌──────┴──────┐                                           │
              │             │                                           │
          Agent Pod    Agent Pod ──► Egress Proxy ──► api.anthropic.com │
          (session A)  (session B)      ▲                               │
              │             │            │                               │
              │     NetworkPolicy: pods can ONLY reach egress-proxy     │
              │                                                         │
            Redis (session → pod-IP mapping)                            │
                     │                                                  │
                     └──────────────────────────────────────────────────┘
```

The agent image is the **same one** Tier 1 builds from
[`hosting/Dockerfile`](../Dockerfile). Same image, different machinery: instead
of a single container or a Modal sandbox, the gateway gives each session its
own pod and the cluster enforces what that pod can reach.

> **Before you self-host:** if you just want a hosted agent without running
> infrastructure, use Anthropic's managed option — see the
> [Hosting overview](../README.md). This guide is for teams that need the
> agent on their own Kubernetes cluster (regulated environments, existing
> platform, custom networking).


## Why each piece exists

**Gateway** — Each user session gets its own agent pod. Something has to create
those pods on demand, route traffic to the right one, and clean them up when
sessions go idle. That's the gateway. It talks to the Kubernetes API to manage
pod lifecycles and uses Redis to remember which session maps to which pod IP.

**Egress proxy + NetworkPolicy** — Agents run arbitrary code. This pair ensures
agent pods can reach `api.anthropic.com` and *nothing else*. The NetworkPolicy
blocks all outbound traffic except to the egress proxy (port 443) and DNS
(port 53). The egress proxy terminates TLS from the agent, then re-encrypts the
request to Anthropic's API. Any attempt to reach the internet, other services,
or other namespaces is dropped at the network level.

**Redis** — The gateway needs to remember which pod is handling which session.
When a request arrives, it looks up the session ID in Redis to find the pod IP
and routes traffic there. Redis persists to disk so mappings survive gateway
restarts.

**Standby pool** — Pods take 10–30 seconds to start (image pull + container
boot). The gateway pre-warms a configurable number of standby pods so new
sessions can claim one instantly instead of waiting. After a pod is claimed,
the pool replenishes in the background.

## Prerequisites

| Tool | What it's for |
|------|---------------|
| [kind](https://kind.sigs.k8s.io/) | Local Kubernetes cluster in Docker |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Applying manifests, inspecting the cluster |
| [docker](https://docs.docker.com/get-docker/) | Building container images |
| `openssl` | Generating the egress proxy's TLS certificate |
| `ANTHROPIC_API_KEY` | Set as env var |

## Quickstart (local, with kind)

```bash
cd hosting/kubernetes
export ANTHROPIC_API_KEY=sk-ant-...
./kind-quickstart.sh
```

This builds the three images, loads them into a local `kind` cluster, applies
every manifest, and port-forwards the gateway to `localhost:8080`. It also
generates bearer tokens for two demo tenants (`alice` and `bob`) and prints
them at the end — export the one you want to use:

```bash
export ALICE_TOKEN=...   # printed by kind-quickstart.sh
export BOB_TOKEN=...
```

## Talk to it

Same path and shape as Tier 1/2 — the base URL changes, and the gateway now
requires a bearer token that identifies the calling tenant:

```bash
curl -N -X POST http://localhost:8080/sessions/demo/messages \
  -H "Authorization: Bearer $ALICE_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"prompt": "What tools do you have?"}'
```

The first request on a new `session_id` claims a standby pod (or spawns one if
the pool is empty). Subsequent requests with the same `session_id` route to the
same pod, so the agent sees a continuous conversation.

The session now belongs to `alice` — the gateway records the creating tenant in
Redis and checks it on every subsequent request. The same call with
`$BOB_TOKEN` returns `403 {"detail":"session belongs to another tenant"}`, and
no token at all returns `401`. The tenant map is a static
`token:tenant,token:tenant` string in the `gateway-tenants` secret; swap the
`authenticate()` stub in [`gateway/main.py`](./gateway/main.py) for your IdP to
derive the tenant from a real credential instead.

Watch the machinery work:

```bash
kubectl -n claude-agent get pods -w
# you'll see agent-standby-* pods appear, then one flip to active when you curl
```

To end a session, go through the gateway so the Redis mapping is cleaned up
(owner-only, like every other session operation):

```bash
curl -X DELETE http://localhost:8080/sessions/demo \
  -H "Authorization: Bearer $ALICE_TOKEN"
```

(`kubectl delete pod` works too, but leaves a stale `session → pod-IP` entry
in Redis until the next request on that session 502s.)

## Verify the egress lockdown

The agent runs code the model decides to run. The egress proxy + NetworkPolicy
mean a prompt-injected agent still can't reach arbitrary hosts. Prove it:

> `kind-quickstart.sh` installs Calico because kind's default CNI (kindnet)
> doesn't enforce NetworkPolicy. On GKE/EKS/AKS or any Calico/Cilium cluster,
> enforcement is on by default and this section works unchanged.

```bash
AGENT_POD=$(kubectl -n claude-agent get pods -l role=agent \
  -o jsonpath='{.items[0].metadata.name}')

# This should FAIL — Calico drops the route to anything except egress-proxy.
# (The agent image is slim and has no curl, so we use Python's socket.)
kubectl -n claude-agent exec "$AGENT_POD" -- python3 -c \
  "import socket; socket.setdefaulttimeout(5); socket.create_connection(('example.com',443)); print('REACHED — policy NOT enforcing')"
```

Expected: `OSError: [Errno 101] Network is unreachable` (or a timeout) and a
non-zero exit. The positive control — that the egress-proxy path *is* open —
was already proven by the curl above returning model output.

## Standby pool

`STANDBY_POOL_SIZE` (in the `agent-config` ConfigMap) controls how many warm
pods the gateway keeps ready. Check current state (any valid tenant token):

```bash
curl http://localhost:8080/api/pool -H "Authorization: Bearer $ALICE_TOKEN"
```

## Persistence

`server.py` persists transcripts (and its caller-ID → SDK-ID map) to
`CLAUDE_CONFIG_DIR=/data`. In this tier that's the pod's ephemeral filesystem,
so:

- **While the pod is alive** (within the idle-timeout window), follow-up
  messages resume the conversation exactly as in Tiers 1 and 2.
- **After the pod is reaped**, `/data` is gone. The next message on that
  `session_id` gets a fresh pod with no history.

For a cookbook demo this is fine — sessions outlive the curl, not the cluster.
For production you need durable storage that survives pod recycle. Two options:

1. **Mount a PersistentVolumeClaim** at `/data` instead of the pod's local
   disk, and have the gateway reattach the same PVC when a session returns.
   Works with `server.py` as-is, but couples each session to a volume in one
   zone.
2. **Mirror `/data` to external storage** with the SDK's
   [`SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage):
   the local-disk write still happens first; the store is a mirror, and
   `mirror_error` is non-fatal. This is the approach the notebook's
   *Making it production-ready* section describes — it needs a small hook in
   `server.py` that the cookbook hasn't grown yet.

## Deploying to your own cluster

`kind` proves the topology; the manifests are cloud-agnostic. To run on EKS,
AKS, GKE, OpenShift, or bare metal, swap the image registry and the front door:

```bash
REG=your.registry.example.com/claude-agent     # ECR, ACR, GHCR, Artifact Registry, ...

# 1. Build and push the three images
docker build -t $REG/agent:latest -f ../Dockerfile ..
docker build -t $REG/gateway:latest ./gateway
docker build -t $REG/egress-proxy:latest ./egress-proxy
docker push $REG/agent:latest $REG/gateway:latest $REG/egress-proxy:latest

# 2. TLS certs for the egress proxy
./generate-certs.sh

# 3. Namespace + secrets + config
kubectl apply -f manifests/namespace.yaml
kubectl -n claude-agent create secret generic anthropic-api-key \
    --from-literal=ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY"
kubectl -n claude-agent create secret generic gateway-tenants \
    --from-literal=GATEWAY_TENANTS="$(openssl rand -hex 16):tenant-a,$(openssl rand -hex 16):tenant-b"
kubectl -n claude-agent create secret generic egress-proxy-tls \
    --from-file=ca.crt=certs/ca.crt \
    --from-file=proxy.crt=certs/proxy.crt \
    --from-file=proxy.key=certs/proxy.key
kubectl -n claude-agent create configmap agent-config \
    --from-literal=AGENT_IMAGE=$REG/agent:latest \
    --from-literal=STANDBY_POOL_SIZE=2

# 4. Apply manifests with your registry substituted
for f in manifests/*.yaml; do
  sed "s|REGISTRY_URL|$REG|g" "$f" | kubectl apply -f -
done
```

> If you later change `$REG`, recreate the `agent-config` ConfigMap as well —
> the gateway reads `AGENT_IMAGE` from it when spawning agent pods, so re-running
> the `sed` over the manifests alone won't repoint them.

Then expose the `gateway` Service through whatever your cluster uses for
ingress — a cloud LoadBalancer, an Ingress controller, or a service mesh
gateway. Three things vary by environment:

- **Registry auth** — your nodes need pull credentials for `$REG`
  (`imagePullSecrets`, IRSA/Workload Identity, or a public registry).
- **NetworkPolicy enforcement** — the egress lockdown only works if your CNI
  enforces `NetworkPolicy` (Cilium, Calico, GKE Dataplane V2, EKS with the
  VPC CNI policy add-on). On a CNI that ignores it, agent pods can reach the
  internet.
- **TLS + auth in front of the gateway** — the static `GATEWAY_TENANTS` token
  map is a stand-in for real credentials. Put your IdP / API gateway in front
  before exposing this publicly.

## What this doesn't give you

- A real identity provider. The gateway *does* enforce per-tenant session
  ownership — each bearer token in `GATEWAY_TENANTS` maps to a tenant, the
  creating tenant owns the session, and other tenants get a 403 — but the
  tokens themselves are a static map with no issuance, rotation, revocation,
  or per-tenant RBAC. Swap `authenticate()` for your IdP; the ownership
  checks don't change.
- Durable session storage (see [Persistence](#persistence))
- Gateway autoscaling or multi-region routing
- DNS-level egress control — port 53 stays open to any resolver so node-local
  DNS caches keep working, which leaves DNS tunneling as a residual
  exfiltration channel. The note in
  [`manifests/network-policy.yaml`](./manifests/network-policy.yaml) shows how
  to tighten it if your cluster talks to kube-dns/CoreDNS directly.
- Hardened supporting services — the gateway, Redis, and nginx run as their
  stock images' default (root) users. The hardening budget went to the agent
  pods, which are the ones running model-driven code; lock down the rest to
  your org's baseline before production.
- Observability beyond what
  [`OTEL_EXPORTER_OTLP_ENDPOINT`](https://code.claude.com/docs/en/agent-sdk/observability)
  gives you for free

## Teardown

```bash
./teardown.sh        # kind delete cluster + remove certs/
```

## Layout

```
kubernetes/
├── README.md
├── kind-quickstart.sh         # local end-to-end on kind
├── teardown.sh
├── generate-certs.sh          # self-signed CA + proxy cert for egress-proxy
├── gateway/
│   ├── main.py                # FastAPI: route + reap
│   ├── k8s.py                 # pod lifecycle + standby pool
│   ├── proxy.py               # SSE relay
│   ├── requirements.txt
│   └── Dockerfile
├── egress-proxy/
│   ├── nginx.conf
│   └── Dockerfile
└── manifests/
    ├── namespace.yaml
    ├── redis.yaml
    ├── egress-proxy.yaml
    ├── gateway.yaml           # SA + RBAC + Deployment + Service
    └── network-policy.yaml
```

---

## 中文翻译

# 第 3 层 — Kubernetes（每个 session 一个 pod）

> 属于 [Agent SDK hosting cookbook](../../07_Hosting_the_agent.ipynb) 的一部分。
> 如果你还没有选定托管层级，请先从那里开始——它会说明何时托管式方案更合适，
> 以及你究竟在什么情况下真的需要这一层。

在 Kubernetes cluster 上运行 agent，其中每个 session 都会获得自己独立的 pod，并通过网络层控制确保 agent pods 只能访问 Anthropic API。

```
                     ┌──────────────────────────────────────────────────┐
                     │                  Kubernetes                      │
                     │                                                  │
  curl / SDK ──────► Gateway (FastAPI)                                  │
                     │  ├─ 通过 K8s API 创建/删除 agent pods            │
                     │  ├─ 将 /sessions/{id}/messages 路由到正确 pod    │
                     │  └─ session → pod 映射存储在 Redis 中            │
                     │                                                  │
              ┌──────┴──────┐                                           │
              │             │                                           │
          Agent Pod    Agent Pod ──► Egress Proxy ──► api.anthropic.com │
          (session A)  (session B)      ▲                               │
              │             │            │                               │
              │     NetworkPolicy：pods 只能访问 egress-proxy           │
              │                                                         │
            Redis（session → pod-IP 映射）                              │
                     │                                                  │
                     └──────────────────────────────────────────────────┘
```

agent 镜像与第 1 层使用的是**同一个镜像**，构建自 [`hosting/Dockerfile`](../Dockerfile)。相同的镜像，不同的运作机制：不是单个容器，也不是 Modal sandbox，而是由 gateway 为每个 session 分配独立 pod，并由 cluster 强制限制该 pod 的网络可达范围。

> **在你自行托管之前：** 如果你只是想要一个托管好的 agent，而不想运行基础设施，请使用 Anthropic 的托管方案——参见 [Hosting overview](../README.md)。本指南面向那些需要将 agent 部署在自有 Kubernetes cluster 上的团队（如受监管环境、已有平台、自定义网络需求）。

## 为什么每个组件都存在

**Gateway** —— 每个用户 session 都会获得自己的 agent pod。必须有某个组件按需创建这些 pod、将流量路由到正确的 pod，并在 session 空闲时清理它们。这就是 gateway 的职责。它通过 Kubernetes API 管理 pod 生命周期，并使用 Redis 记录 session 与 pod IP 的映射关系。

**Egress proxy + NetworkPolicy** —— Agents 会运行任意代码。这一组合确保 agent pods 只能访问 `api.anthropic.com`，并且*不能访问其他任何地方*。`NetworkPolicy` 会阻止除 egress proxy（443 端口）和 DNS（53 端口）之外的所有出站流量。egress proxy 接收来自 agent 的 TLS 连接，然后重新加密请求并转发给 Anthropic API。任何访问互联网、其他服务或其他 namespace 的尝试，都会在网络层被丢弃。

**Redis** —— Gateway 需要记住哪个 pod 正在处理哪个 session。当请求到达时，它会在 Redis 中查找 session ID 对应的 pod IP，并将流量转发到那里。Redis 会持久化到磁盘，因此即使 gateway 重启，映射关系也能保留。

**Standby pool** —— Pods 启动通常需要 10–30 秒（拉取镜像 + 容器启动）。Gateway 会预热一定数量的 standby pods，这样新 session 可以立即领取一个，而不必等待。某个 pod 被领取后，池子会在后台补齐。

## 前置条件

| Tool | 用途 |
|------|------|
| [kind](https://kind.sigs.k8s.io/) | 在 Docker 中运行本地 Kubernetes cluster |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | 应用 manifests、检查 cluster |
| [docker](https://docs.docker.com/get-docker/) | 构建容器镜像 |
| `openssl` | 生成 egress proxy 的 TLS 证书 |
| `ANTHROPIC_API_KEY` | 通过环境变量设置 |

## 快速开始（本地，使用 kind）

```bash
cd hosting/kubernetes
export ANTHROPIC_API_KEY=sk-ant-...
./kind-quickstart.sh
```

这会构建三个镜像，将它们加载到本地 `kind` cluster 中，应用所有 manifest，并将 gateway 端口转发到 `localhost:8080`。它还会为两个演示租户（`alice` 和 `bob`）生成 bearer token，并在最后打印出来——导出你要使用的那个：

```bash
export ALICE_TOKEN=...   # 由 kind-quickstart.sh 打印
export BOB_TOKEN=...
```

## 如何与其交互

路径与请求格式与第 1/2 层相同——只是 base URL 改变了，并且现在 gateway 需要一个用于标识调用租户的 bearer token：

```bash
curl -N -X POST http://localhost:8080/sessions/demo/messages \
  -H "Authorization: Bearer $ALICE_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"prompt": "What tools do you have?"}'
```

对新 `session_id` 的第一次请求会领取一个 standby pod（如果池为空则创建一个）。后续使用相同 `session_id` 的请求会被路由到同一个 pod，因此 agent 能看到连续的会话上下文。

该 session 现在归 `alice` 所有——gateway 会将创建者租户记录在 Redis 中，并在之后的每个请求中进行校验。相同调用若改用 `$BOB_TOKEN`，将返回 `403 {"detail":"session belongs to another tenant"}`；完全不带 token 则返回 `401`。租户映射是 `gateway-tenants` secret 中的静态 `token:tenant,token:tenant` 字符串；要接入真实身份系统，可将 [`gateway/main.py`](./gateway/main.py) 中的 `authenticate()` stub 替换为你的 IdP，从真实凭证中派生租户身份。

观察这些机制如何工作：

```bash
kubectl -n claude-agent get pods -w
# 你会看到 agent-standby-* pods 出现，然后在执行 curl 时其中一个切换为 active
```

要结束一个 session，请通过 gateway 进行，以便清理 Redis 中的映射关系（仅所有者可操作，和其他 session 操作一致）：

```bash
curl -X DELETE http://localhost:8080/sessions/demo \
  -H "Authorization: Bearer $ALICE_TOKEN"
```

（`kubectl delete pod` 也可以，但会在 Redis 中留下陈旧的 `session → pod-IP` 记录，直到该 session 的下一次请求返回 502。）

## 验证出站访问锁定

agent 会运行由模型决定执行的代码。egress proxy + NetworkPolicy 意味着即使 prompt 注入成功，agent 仍然无法访问任意主机。你可以自己验证：

> `kind-quickstart.sh` 会安装 Calico，因为 kind 默认的 CNI（kindnet）不执行 `NetworkPolicy`。在 GKE/EKS/AKS 或任何 Calico/Cilium cluster 上，默认就会执行，本节内容可原样适用。

```bash
AGENT_POD=$(kubectl -n claude-agent get pods -l role=agent \
  -o jsonpath='{.items[0].metadata.name}')

# 这应该失败 —— Calico 会丢弃除 egress-proxy 之外的所有路由。
# （agent 镜像较精简，没有 curl，因此我们使用 Python 的 socket。）
kubectl -n claude-agent exec "$AGENT_POD" -- python3 -c \
  "import socket; socket.setdefaulttimeout(5); socket.create_connection(('example.com',443)); print('REACHED — policy NOT enforcing')"
```

预期结果：出现 `OSError: [Errno 101] Network is unreachable`（或者超时），并返回非零退出码。正向验证——即 egress-proxy 路径确实是畅通的——已经通过前面的 curl 返回模型输出得到证明。

## Standby pool

`STANDBY_POOL_SIZE`（位于 `agent-config` ConfigMap 中）控制 gateway 预先保留多少个 warm pods。查看当前状态（任意有效租户 token 均可）：

```bash
curl http://localhost:8080/api/pool -H "Authorization: Bearer $ALICE_TOKEN"
```

## 持久化

`server.py` 会将对话记录（以及它的 caller-ID → SDK-ID 映射）持久化到 `CLAUDE_CONFIG_DIR=/data`。在这一层中，这个路径位于 pod 的临时文件系统中，因此：

- **只要 pod 还活着**（处于 idle-timeout 窗口内），后续消息就会像第 1 和第 2 层那样准确恢复对话。
- **一旦 pod 被回收**，`/data` 就会消失。对同一 `session_id` 的下一条消息会获得一个没有历史记录的新 pod。

对于 cookbook 演示来说这已经足够——session 的生命周期比一次 curl 更长，但不要求比 cluster 更长。若用于生产环境，你需要在 pod 回收后仍能保留数据的持久存储。有两种方案：

1. 在 `/data` 挂载 **PersistentVolumeClaim**，替代 pod 本地磁盘，并在 session 返回时由 gateway 重新挂载同一个 PVC。这样 `server.py` 无需修改即可工作，但每个 session 会绑定到某个可用区中的一个 volume。
2. 使用 SDK 的 [`SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage) 将 `/data` **镜像到外部存储**：本地磁盘写入仍然先发生；store 只是镜像，`mirror_error` 不会导致失败。这也是 notebook 中 *Making it production-ready* 一节描述的方法——但 cookbook 目前还没有在 `server.py` 中加上这个小 hook。

## 部署到你自己的 cluster

`kind` 用于验证整体拓扑；这些 manifests 与云厂商无关。若要部署到 EKS、AKS、GKE、OpenShift 或裸金属环境，需要替换镜像仓库和入口层：

```bash
REG=your.registry.example.com/claude-agent     # ECR, ACR, GHCR, Artifact Registry, ...

# 1. 构建并推送三个镜像
docker build -t $REG/agent:latest -f ../Dockerfile ..
docker build -t $REG/gateway:latest ./gateway
docker build -t $REG/egress-proxy:latest ./egress-proxy
docker push $REG/agent:latest $REG/gateway:latest $REG/egress-proxy:latest

# 2. 为 egress proxy 生成 TLS 证书
./generate-certs.sh

# 3. Namespace + secrets + config
kubectl apply -f manifests/namespace.yaml
kubectl -n claude-agent create secret generic anthropic-api-key \
    --from-literal=ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY"
kubectl -n claude-agent create secret generic gateway-tenants \
    --from-literal=GATEWAY_TENANTS="$(openssl rand -hex 16):tenant-a,$(openssl rand -hex 16):tenant-b"
kubectl -n claude-agent create secret generic egress-proxy-tls \
    --from-file=ca.crt=certs/ca.crt \
    --from-file=proxy.crt=certs/proxy.crt \
    --from-file=proxy.key=certs/proxy.key
kubectl -n claude-agent create configmap agent-config \
    --from-literal=AGENT_IMAGE=$REG/agent:latest \
    --from-literal=STANDBY_POOL_SIZE=2

# 4. 将你的 registry 替换进 manifests 并应用
for f in manifests/*.yaml; do
  sed "s|REGISTRY_URL|$REG|g" "$f" | kubectl apply -f -
done
```

> 如果之后你修改了 `$REG`，还需要重新创建 `agent-config` ConfigMap——gateway 在创建 agent pods 时会从中读取 `AGENT_IMAGE`，所以仅仅重新对 manifests 执行 `sed` 并不能让它们指向新镜像。

然后，通过你的 cluster 所使用的入口方案暴露 `gateway` Service——例如云厂商的 LoadBalancer、Ingress controller 或 service mesh gateway。不同环境下有三点会变化：

- **Registry auth** —— 你的节点需要具备拉取 `$REG` 的凭据（`imagePullSecrets`、IRSA/Workload Identity，或使用公共 registry）。
- **NetworkPolicy enforcement** —— 出站访问锁定只有在你的 CNI 执行 `NetworkPolicy` 时才有效（如 Cilium、Calico、GKE Dataplane V2、启用了 VPC CNI policy add-on 的 EKS）。如果 CNI 忽略它，agent pods 仍可访问互联网。
- **Gateway 前的 TLS + 认证** —— 静态的 `GATEWAY_TENANTS` token 映射只是对真实凭证系统的一个替身。在公开暴露之前，请在前面接入你的 IdP / API gateway。

## 这套方案不包含什么

- 真实的身份提供方。Gateway *确实会*执行按租户隔离的 session 所有权校验——`GATEWAY_TENANTS` 中的每个 bearer token 都映射到一个租户，创建 session 的租户拥有该 session，其他租户会收到 403——但这些 token 本身只是静态映射，不具备签发、轮换、撤销或按租户 RBAC。将 `authenticate()` 替换为你的 IdP 后，所有权校验逻辑本身无需改变。
- 持久化 session 存储（参见 [Persistence](#persistence)）
- Gateway 自动伸缩或多区域路由
- DNS 层级的出站控制——53 端口会对任何 resolver 保持开放，以便 node-local DNS caches 正常工作，这仍然留下了 DNS 隧道作为残余数据外传通道。在 [`manifests/network-policy.yaml`](./manifests/network-policy.yaml) 中的说明展示了如果你的 cluster 直接与 kube-dns/CoreDNS 通信，应如何进一步收紧。
- 加固过的配套服务——gateway、Redis 和 nginx 都以其 stock images 的默认（root）用户运行。加固预算优先投入到了 agent pods，因为它们才是运行模型驱动代码的组件；在投入生产前，请按你们组织的基线对其他部分进行加固。
- 除了 [`OTEL_EXPORTER_OTLP_ENDPOINT`](https://code.claude.com/docs/en/agent-sdk/observability) 免费提供的内容之外，更进一步的可观测性能力

## 清理

```bash
./teardown.sh        # kind delete cluster + remove certs/
```

## 布局

```
kubernetes/
├── README.md
├── kind-quickstart.sh         # 在 kind 上跑通本地端到端
├── teardown.sh
├── generate-certs.sh          # 为 egress-proxy 生成自签名 CA + proxy 证书
├── gateway/
│   ├── main.py                # FastAPI：路由 + 回收
│   ├── k8s.py                 # pod 生命周期 + standby pool
│   ├── proxy.py               # SSE 转发
│   ├── requirements.txt
│   └── Dockerfile
├── egress-proxy/
│   ├── nginx.conf
│   └── Dockerfile
└── manifests/
    ├── namespace.yaml
    ├── redis.yaml
    ├── egress-proxy.yaml
    ├── gateway.yaml           # SA + RBAC + Deployment + Service
    └── network-policy.yaml
```
