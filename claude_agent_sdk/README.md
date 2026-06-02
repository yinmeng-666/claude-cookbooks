# Building Powerful Agents with the Claude Agent SDK

A tutorial series demonstrating how to build sophisticated general-purpose agentic systems using the [Claude Agent SDK](https://github.com/anthropics/claude-agent-sdk-python), progressing from simple research agents to multi-agent orchestration with external system integration.

## Getting Started

#### 1. Install uv, [node](https://nodejs.org/en/download/), and the Claude Code CLI (if you haven't already)

```curl -LsSf https://astral.sh/uv/install.sh | sh ```

```npm install -g @anthropic-ai/claude-code```

#### 2. Clone and set up the project

```git clone https://github.com/anthropics/anthropic-cookbook.git ```

```cd anthropic-cookbook/claude_agent_sdk```

```uv sync ```

#### 3. Register venv as Jupyter kernel so that you can use it in the notebooks

```uv run python -m ipykernel install --user --name="cc-sdk-tutorial" --display-name "Python (cc-sdk-tutorial)" ```

#### 4. Claude API Key
1. Visit [platform.claude.ai](https://platform.claude.ai/dashboard)
2. Sign up or log in to your account
3. Click on "Get API keys"
4. Copy the key and paste it into your `.env` file as ```ANTHROPIC_API_KEY=```

#### 5. GitHub Token for Notebook 02
If you plan to work through the Observability Agent notebook:
1. Get a GitHub Personal Access Token [here](https://github.com/settings/personal-access-tokens/new)
2. Select "Fine-grained" token with default options (public repos, no account permissions)
3. Add it to your `.env` file as `GITHUB_TOKEN="<token>"`
4. Ensure [Docker](https://www.docker.com/products/docker-desktop/) is running on your machine

## Tutorial Series Overview

This tutorial series takes you on a journey from basic agent implementation to sophisticated multi-agent systems capable of handling real-world complexity. Each notebook builds upon the previous one, introducing new concepts and capabilities while maintaining practical, production-ready implementations.

### What You'll Learn

Through this series, you'll be exposed to:
- **Core SDK fundamentals** with `query()` and the `ClaudeSDKClient` & `ClaudeAgentOptions` interfaces in the Python SDK
- **Tool usage patterns** from basic WebSearch to complex MCP server integration
- **Multi-agent orchestration** with specialized subagents and coordination
- **Enterprise features** by leveraging hooks for compliance tracking and audit trails
- **External system integration** via Model Context Protocol (MCP)

Note: This tutorial assumes you have some level of familiarity with Claude Code. Ideally, if you have been using Claude Code to supercharge your coding tasks and would like to leverage its raw agentic power for tasks beyond Software Engineering, this tutorial will help you get started.

## Notebook Structure & Content

### [Notebook 00: The One-Liner Research Agent](00_The_one_liner_research_agent.ipynb)

Start your journey with a simple yet powerful research agent built in just a few lines of code. This notebook introduces core SDK concepts and demonstrates how the Claude Agent SDK enables autonomous information gathering and synthesis.

**Key Concepts:**
- Basic agent loops with `query()` and async iteration
- WebSearch tool for autonomous research
- Multimodal capabilities with the Read tool
- Conversation context management with `ClaudeSDKClient`
- System prompts for agent specialization

### [Notebook 01: The Chief of Staff Agent](01_The_chief_of_staff_agent.ipynb)

Build a comprehensive AI Chief of Staff for a startup CEO, showcasing advanced SDK features for production environments. This notebook demonstrates how to create sophisticated agent architectures with governance, compliance, and specialized expertise.

**Key Features Explored:**
- **Memory & Context:** Persistent instructions with CLAUDE.md files
- **Output Styles:** Tailored communication for different audiences
- **Plan Mode:** Strategic planning without execution for complex tasks
- **Custom Slash Commands:** User-friendly shortcuts for common operations
- **Hooks:** Automated compliance tracking and audit trails
- **Subagent Orchestration:** Coordinating specialized agents for domain expertise
- **Bash Tool Integration:** Python script execution for procedural knowledge and complex computations

### [Notebook 02: The Observability Agent](02_The_observability_agent.ipynb)

Expand beyond local capabilities by connecting agents to external systems through the Model Context Protocol. Transform your agent from a passive observer into an active participant in DevOps workflows.

**Advanced Capabilities:**
- **Git MCP Server:** 13+ tools for repository analysis and version control
- **GitHub MCP Server:** 100+ tools for complete GitHub platform integration
- **Real-time Monitoring:** CI/CD pipeline analysis and failure detection
- **Intelligent Incident Response:** Automated root cause analysis
- **Production Workflow Automation:** From monitoring to actionable insights

### [Notebook 03: The Site Reliability Agent](03_The_site_reliability_agent.ipynb)

Move from read-only observation to read-write remediation. Build an SRE incident response agent that can investigate production incidents, diagnose root causes, apply fixes, and document the results — all autonomously.

**Key Capabilities:**
- **MCP Tool Server:** 12+ tools for metrics, infrastructure, diagnostics, and documentation via JSON-RPC subprocess
- **Prometheus Integration:** PromQL queries for error rates, latency, and DB connection monitoring
- **Read-Write Remediation:** Edit configuration files, restart Docker services, and verify fixes
- **Safety Hooks:** PreToolUse hooks that validate write operations (pool size ranges, config sanity checks)
- **End-to-End Incident Lifecycle:** From detection through remediation to post-mortem documentation
- **Production Extensions:** Optional PagerDuty and Confluence integrations via conditional MCP tool registration

## Complete Agent Implementations

Each notebook includes an agent implementation in its respective directory:
- **`research_agent/`** - Autonomous research agent with web search and multimodal analysis
- **`chief_of_staff_agent/`** - Multi-agent executive assistant with financial modeling and compliance
- **`observability_agent/`** - DevOps monitoring agent with GitHub integration
- **`site_reliability_agent/`** - SRE incident response agent with Prometheus, Docker, and MCP tool server

**Running standalone agents:** To import agent modules outside of notebooks, either run from the `claude_agent_sdk/` directory or install the package in editable mode:
```bash
uv pip install -e .
```

## Background
### The Evolution of Claude Agent SDK

Claude Code has emerged as one of Anthropic's most successful products, but not just for its SOTA coding capabilities. Its true breakthrough lies in something more fundamental: **Claude is exceptionally good at agentic work**.

What makes Claude Code special isn't just code understanding; it's the ability to:
- Break down complex tasks into manageable steps autonomously
- Use tools effectively and make intelligent decisions about which tools to use and when
- Maintain context and memory across long-running tasks
- Recover gracefully from errors and adapt approaches when needed
- Know when to ask for clarification versus when to proceed with reasonable assumptions

These capabilities have made Claude Code the closest thing to a "bare metal" harness for Claude's raw agentic power: a minimal yet complete and sophisticated interface that lets the model's capabilities shine with the least possible overhead.

### Beyond Coding: The Agent Builder's Toolkit

Originally an internal tool built by Anthropic engineers to accelerate development workflows, the SDK's public release revealed unexpected potential. After the release of the Claude Agent SDK and its GitHub integration, developers began using it for tasks far beyond coding:

- **Research agents** that gather and synthesize information across multiple sources
- **Data analysis agents** that explore datasets and generate insights
- **Workflow automation agents** that handle repetitive business processes
- **Monitoring and observability agents** that watch systems and respond to issues
- **Content generation agents** that create and refine various types of content

The pattern was clear: the SDK had inadvertently become an effective agent-building framework. Its architecture, designed to handle software development complexity, proved remarkably well-suited for general-purpose agent creation.

This tutorial series demonstrates how to leverage the Claude Agent SDK to build highly efficient agents for any domain or use case, from simple automation to complex enterprise systems. 

## Contributing

Found an issue or have a suggestion? Please open an issue or submit a pull request!

---

## 中文翻译

一个教程系列，演示如何使用 [Claude Agent SDK](https://github.com/anthropics/claude-agent-sdk-python) 构建复杂的通用智能体系统，从简单的研究型智能体逐步进阶到集成外部系统的多智能体编排。

## 快速开始

#### 1. 安装 `uv`、[node](https://nodejs.org/en/download/) 和 Claude Code CLI（如果你还没有安装）

```curl -LsSf https://astral.sh/uv/install.sh | sh ```
```npm install -g @anthropic-ai/claude-code```

#### 2. 克隆并设置项目

```git clone https://github.com/anthropics/anthropic-cookbook.git ```
```cd anthropic-cookbook/claude_agent_sdk```
```uv sync ```

#### 3. 将 `venv` 注册为 Jupyter kernel，以便在 notebooks 中使用

```uv run python -m ipykernel install --user --name="cc-sdk-tutorial" --display-name "Python (cc-sdk-tutorial)" ```

#### 4. Claude API Key
1. 访问 [platform.claude.ai](https://platform.claude.ai/dashboard)
2. 注册或登录你的账号
3. 点击 “Get API keys”
4. 复制 key，并将其粘贴到你的 `.env` 文件中，格式为 ```ANTHROPIC_API_KEY=```

#### 5. 用于 Notebook 02 的 GitHub Token
如果你打算学习 Observability Agent notebook：
1. 在[这里](https://github.com/settings/personal-access-tokens/new)获取一个 GitHub Personal Access Token
2. 选择 “Fine-grained” token，并使用默认选项（public repos，无 account permissions）
3. 将其添加到你的 `.env` 文件中，格式为 `GITHUB_TOKEN="<token>"`
4. 确保你的机器上已经运行 [Docker](https://www.docker.com/products/docker-desktop/)

## 教程系列概览

本教程系列将带你完成从基础智能体实现到复杂多智能体系统的完整进阶过程，这些系统能够处理真实世界中的复杂问题。每个 notebook 都建立在前一个的基础之上，在保持实践性和可用于生产环境的实现方式的同时，引入新的概念与能力。

### 你将学到什么

通过这个系列，你将接触到：
- 使用 Python SDK 中的 `query()` 以及 `ClaudeSDKClient` 和 `ClaudeAgentOptions` 接口的 **SDK 核心基础**
- 从基础 `WebSearch` 到复杂 MCP server 集成的 **工具使用模式**
- 通过专门子智能体与协调机制实现的 **多智能体编排**
- 利用 hooks 实现合规跟踪和审计链路的 **企业级特性**
- 通过 Model Context Protocol (MCP) 实现的 **外部系统集成**

注意：本教程默认你对 Claude Code 有一定程度的熟悉。理想情况下，如果你已经在使用 Claude Code 来增强你的编码工作，并希望将其原生的智能体能力扩展到软件工程之外的任务，那么本教程将帮助你快速上手。

## Notebook 结构与内容

### [Notebook 00: The One-Liner Research Agent](00_The_one_liner_research_agent.ipynb)

从一个简单但强大的研究型智能体开始你的学习之旅，它只需几行代码即可构建完成。这个 notebook 介绍了 SDK 的核心概念，并展示 Claude Agent SDK 如何实现自主的信息收集与综合。

**关键概念：**
- 使用 `query()` 和异步迭代实现基础智能体循环
- 用于自主研究的 `WebSearch` 工具
- 结合 `Read` 工具的多模态能力
- 使用 `ClaudeSDKClient` 进行会话上下文管理
- 用于智能体专门化的 system prompts

### [Notebook 01: The Chief of Staff Agent](01_The_chief_of_staff_agent.ipynb)

为一家创业公司的 CEO 构建一个全面的 AI Chief of Staff，展示适用于生产环境的高级 SDK 特性。这个 notebook 演示了如何构建具备治理、合规与专业能力的复杂智能体架构。

**探索的关键特性：**
- **Memory & Context：** 使用 `CLAUDE.md` 文件提供持久化指令
- **Output Styles：** 面向不同受众定制沟通方式
- **Plan Mode：** 针对复杂任务进行只规划不执行的战略规划
- **Custom Slash Commands：** 为常见操作提供易用的快捷命令
- **Hooks：** 自动化合规跟踪与审计链路
- **Subagent Orchestration：** 协调专门智能体以提供领域专长
- **Bash Tool Integration：** 通过 Python 脚本执行实现程序性知识与复杂计算

### [Notebook 02: The Observability Agent](02_The_observability_agent.ipynb)

通过 Model Context Protocol 将智能体连接到外部系统，从而突破本地能力的限制。把你的智能体从被动观察者转变为 DevOps 工作流中的主动参与者。

**高级能力：**
- **Git MCP Server：** 13+ 个工具，用于仓库分析和版本控制
- **GitHub MCP Server：** 100+ 个工具，用于完整的 GitHub 平台集成
- **Real-time Monitoring：** CI/CD pipeline 分析与故障检测
- **Intelligent Incident Response：** 自动化根因分析
- **Production Workflow Automation：** 从监控到可执行洞察的全流程自动化

### [Notebook 03: The Site Reliability Agent](03_The_site_reliability_agent.ipynb)

从只读观察迈向读写修复。构建一个 SRE 事故响应智能体，它可以自主调查生产事故、诊断根因、应用修复并记录结果。

**关键能力：**
- **MCP Tool Server：** 12+ 个工具，通过 JSON-RPC subprocess 提供指标、基础设施、诊断和文档能力
- **Prometheus Integration：** 使用 PromQL 查询错误率、延迟和 DB connection 监控
- **Read-Write Remediation：** 编辑配置文件、重启 Docker services 并验证修复效果
- **Safety Hooks：** `PreToolUse` hooks，用于验证写操作（连接池大小范围、配置合理性检查）
- **End-to-End Incident Lifecycle：** 从检测、修复到事后复盘文档的全流程
- **Production Extensions：** 通过条件式 MCP tool registration 可选集成 PagerDuty 和 Confluence

## 完整的智能体实现

每个 notebook 都在其对应目录中包含一个智能体实现：
- **`research_agent/`** - 带有 web search 和多模态分析的自主研究型智能体
- **`chief_of_staff_agent/`** - 带有财务建模与合规能力的多智能体高管助理
- **`observability_agent/`** - 集成 GitHub 的 DevOps 监控智能体
- **`site_reliability_agent/`** - 集成 Prometheus、Docker 和 MCP tool server 的 SRE 事故响应智能体

**运行独立智能体：** 若要在 notebooks 之外导入智能体模块，请在 `claude_agent_sdk/` 目录下运行，或者以 editable mode 安装该包：

```bash
uv pip install -e .
```

## 背景
### Claude Agent SDK 的演进

Claude Code 已成为 Anthropic 最成功的产品之一，但这并不仅仅因为它具备 SOTA 级别的编码能力。它真正的突破点在于更基础的一件事：**Claude 在智能体式工作方面表现得异常出色**。

Claude Code 的特别之处并不只是代码理解能力，而在于它能够：
- 自主地将复杂任务拆解为可管理的步骤
- 高效使用工具，并能智能判断该使用什么工具以及何时使用
- 在长时间运行的任务中保持上下文与记忆
- 从错误中优雅恢复，并在需要时调整方法
- 知道何时需要请求澄清，何时应基于合理假设继续推进

这些能力使 Claude Code 成为了最接近 Claude 原始智能体能力“bare metal harness”的存在：一个最小化但完整且精巧的接口，让模型能力能够以尽可能低的额外开销得到充分发挥。

### 超越编码：智能体构建者的工具箱

SDK 最初是 Anthropic 工程师为加速开发工作流而构建的内部工具，但其公开发布后展现出了意料之外的潜力。在 Claude Agent SDK 及其 GitHub 集成发布后，开发者开始将其用于远超编码范围的任务：

- 跨多个来源收集并综合信息的 **研究型智能体**
- 探索数据集并生成洞察的 **数据分析智能体**
- 处理重复性业务流程的 **工作流自动化智能体**
- 监控系统并对问题作出响应的 **监控与可观测性智能体**
- 创建并打磨各类内容的 **内容生成智能体**

这种模式已经很清晰：SDK 在无意中变成了一个高效的智能体构建框架。它原本为处理软件开发复杂性而设计的架构，被证明也非常适合构建通用型智能体。

本教程系列将展示如何利用 Claude Agent SDK 为任意领域和使用场景构建高效智能体，从简单自动化到复杂企业系统皆可覆盖。

## 贡献

发现问题或者有建议吗？欢迎提交 issue 或 pull request！
