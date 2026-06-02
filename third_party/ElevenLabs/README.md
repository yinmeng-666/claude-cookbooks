# ElevenLabs <> Claude Cookbooks

[ElevenLabs](https://elevenlabs.io/) provides AI-powered speech-to-text and text-to-speech APIs for creating natural-sounding voice applications with advanced features like voice cloning and streaming synthesis.

This cookbook demonstrates how to build a low-latency voice assistant by combining ElevenLabs' speech processing with Claude's intelligent responses, progressively optimizing for real-time performance.

## What's Included

* **[Low Latency Voice Assistant Notebook](./low_latency_stt_claude_tts.ipynb)** - An interactive tutorial that walks you through building a voice assistant step-by-step, demonstrating various optimization techniques to minimize latency through streaming.

* **[WebSocket Streaming Script](./stream_voice_assistant_websocket.py)** - A production-ready conversational voice assistant featuring continuous microphone input, gapless audio playback, and the lowest possible latency using WebSocket streaming.

## How to Use This Cookbook

We recommend following this sequence to get the most out of this cookbook:

### Step 1: Set Up Your Environment

1. **Create a virtual environment:**
   ```bash
   # Navigate to the ElevenLabs directory
   cd /path/to/claude-cookbooks/third_party/ElevenLabs

   # Create virtual environment
   python -m venv venv

   # Activate it
   source venv/bin/activate  # On macOS/Linux
   # OR
   venv\Scripts\activate     # On Windows
   ```

2. **Get your API keys:**
   - **ElevenLabs API key:** [elevenlabs.io/app/developers/api-keys](https://elevenlabs.io/app/developers/api-keys)

     When creating your API key, ensure it has the following minimum permissions:
     - Text to speech
     - Speech to text
     - Read access on voices
     - Read access on models

   - **Anthropic API key:** [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)

3. **Configure your environment:**
   ```bash
   cp .env.example .env
   ```

   Edit `.env` and add your API keys:
   ```
   ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
   ANTHROPIC_API_KEY=sk-ant-api03-...
   ```

4. **Install dependencies:**
   ```bash
   # With venv activated
   pip install -r requirements.txt
   ```

### Step 2: Work Through the Notebook

Start with the **[Low Latency Voice Assistant Notebook](./low_latency_stt_claude_tts.ipynb)**. This interactive guide will teach you:

- How to use ElevenLabs for speech-to-text transcription
- How to generate Claude responses and measure latency
- How streaming reduces time-to-first-token
- How to stream text-to-speech for faster audio playback
- The tradeoffs between different streaming approaches
- Why WebSocket streaming provides the best balance of latency and quality

The notebook includes performance metrics and comparisons at each step, helping you understand the impact of each optimization.

### Step 3: Try the Production Script

After understanding the concepts from the notebook, run the **[WebSocket Streaming Script](./stream_voice_assistant_websocket.py)** to experience a fully functional voice assistant:

```bash
python stream_voice_assistant_websocket.py
```

**How it works:**
1. Press Enter to start recording
2. Speak your question into the microphone
3. Press Enter to stop recording
4. The assistant will respond with natural speech
5. Repeat or press Ctrl+C to exit

The script demonstrates production-ready implementations of:
- Real-time microphone recording with sounddevice
- Continuous conversation with context retention
- WebSocket-based streaming for minimal latency
- Custom audio queue for seamless playback

## Troubleshooting

### Audio Popping or Crackling

**Symptom:** You may occasionally hear brief pops, clicks, or audio dropouts during playback.

**Explanation:**

This occurs because the script uses MP3 format audio, which is required for the ElevenLabs free tier. When streaming MP3 data in real-time chunks, FFmpeg occasionally receives incomplete frames that cannot be decoded. This typically happens:
- At the start of streaming (first chunk may be too small)
- During brief network delays
- At the end of audio generation (final chunk may be partial)

The script automatically handles these failed chunks by skipping them (using a try-except pattern in the audio decoding logic), which prevents errors from appearing in the console but may result in brief audio gaps that manifest as pops or clicks.

**Impact:**
- Audio playback continues normally
- Brief pops or clicks are usually imperceptible or minor
- The WebSocket connection remains stable
- No functionality is lost

**Solution:**

This is expected behavior when using MP3 format on the free tier. If you want to eliminate audio popping entirely:
1. Upgrade to a paid ElevenLabs tier
2. Modify the script to use `pcm_44100` format instead of MP3
3. PCM format provides cleaner streaming without decoding issues

### API Key Issues

**Symptom:** `AssertionError: ELEVENLABS_API_KEY is not set` or `AssertionError: ANTHROPIC_API_KEY is not set`

**Solution:**
1. Verify you've copied `.env.example` to `.env`: `cp .env.example .env`
2. Edit `.env` and ensure both API keys are set correctly
3. Check for typos or extra spaces in your API keys
4. Confirm your ElevenLabs key has the required permissions (see Step 1)

### Dependency Issues

**Symptom:** Errors like `ImportError: PortAudio library not found` or audio playback failures

**Solution:**

**macOS:**
```bash
brew install portaudio ffmpeg
```

**Ubuntu/Debian:**
```bash
sudo apt-get install portaudio19-dev ffmpeg
```

**Windows:**
- Install FFmpeg from [ffmpeg.org](https://ffmpeg.org/download.html)
- Add FFmpeg to your system PATH
- PortAudio typically installs automatically with sounddevice on Windows

Then reinstall Python dependencies:
```bash
pip install -r requirements.txt
```

### Microphone Permissions

**Symptom:** `OSError: [Errno -9999] Unanticipated host error` or microphone not accessible

**Solution:**
- **macOS:** Go to System Preferences → Security & Privacy → Privacy → Microphone, and enable Terminal (or your Python IDE)
- **Windows:** Go to Settings → Privacy → Microphone, and enable microphone access for Python/Terminal
- **Linux:** Check your user is in the `audio` group: `sudo usermod -a -G audio $USER` (then log out and back in)

Test your microphone setup:
```bash
python -c "import sounddevice as sd; print(sd.query_devices())"
```

### WebSocket Connection Failures

**Symptom:** Connection errors, timeouts, or stream interruptions

**Solution:**
1. Check your internet connection is stable
2. Verify firewall isn't blocking WebSocket connections (port 443)
3. Try disabling VPN or proxy temporarily
4. Ensure you're not exceeding API rate limits (see ElevenLabs dashboard for usage)

If you continue to experience issues, check [ElevenLabs Status](https://status.elevenlabs.io/) for service updates.

## Project Ideas

Once you're comfortable with the voice assistant, here are some inspiring projects you can build:

- **Meeting Note-Taker** - Record and transcribe meetings in real-time, then use Claude to generate summaries, action items, and key takeaways from the conversation.

- **Language Learning Tutor** - Practice conversations in any language with real-time feedback. Claude can correct pronunciation, suggest better phrasing, and adapt difficulty to your skill level.

- **Interactive Storyteller** - Create choose-your-own-adventure games where Claude narrates the story and responds to your spoken choices, with different voice characters for each role.

- **Hands-Free Coding Assistant** - Describe code changes, bugs, or features verbally while keeping your hands on the keyboard. Perfect for rubber duck debugging or pair programming solo.

- **Voice-Activated Smart Home** - Build natural conversation interfaces for controlling home devices. Ask complex questions like "Is it cold enough to turn on the heater?" instead of simple on/off commands.

- **Personal Voice Journal** - Keep a daily journal by speaking your thoughts. Claude can organize entries by theme, track your mood over time, and surface relevant past entries when you need them.

## More About ElevenLabs

Here are some helpful resources to deepen your understanding:

- [ElevenLabs Platform](https://elevenlabs.io/) - Official website
- [API Documentation](https://elevenlabs.io/docs/overview) - Complete API reference
- [Voice Library](https://elevenlabs.io/voice-library) - Explore available voices
- [API Playground](https://elevenlabs.io/app/speech-synthesis/text-to-speech) - Test voices interactively
- [Python SDK](https://github.com/elevenlabs/elevenlabs-python) - Official Python SDK

---

## 中文翻译

# ElevenLabs &lt;&gt; Claude Cookbooks

[ElevenLabs](https://elevenlabs.io/) 提供由 AI 驱动的语音转文本和文本转语音 API，可用于创建自然发声的语音应用，并支持语音克隆、流式合成等高级能力。

本 cookbook 展示了如何将 ElevenLabs 的语音处理与 Claude 的智能响应结合起来构建一个低延迟语音助手，并逐步针对实时性能进行优化。

## 包含内容

* **[低延迟语音助手 Notebook](./low_latency_stt_claude_tts.ipynb)** - 一个交互式教程，手把手带你构建语音助手，并演示多种通过流式处理将延迟降到最低的优化技术。

* **[WebSocket 流式脚本](./stream_voice_assistant_websocket.py)** - 一个可用于生产的对话式语音助手，具备持续麦克风输入、无缝音频播放以及通过 WebSocket 流式处理实现的最低可能延迟。

## 如何使用本 Cookbook

我们建议按以下顺序进行，以最大化利用本 cookbook：

### 第 1 步：设置你的环境

1. **创建虚拟环境：**
   ```bash
   # Navigate to the ElevenLabs directory
   cd /path/to/claude-cookbooks/third_party/ElevenLabs

   # Create virtual environment
   python -m venv venv

   # Activate it
   source venv/bin/activate  # On macOS/Linux
   # OR
   venv\Scripts\activate     # On Windows
   ```

2. **获取你的 API keys：**
   - **ElevenLabs API key：** [elevenlabs.io/app/developers/api-keys](https://elevenlabs.io/app/developers/api-keys)

     创建 API key 时，请确保它至少具备以下权限：
     - Text to speech
     - Speech to text
     - Read access on voices
     - Read access on models

   - **Anthropic API key：** [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)

3. **配置你的环境：**
   ```bash
   cp .env.example .env
   ```

   编辑 `.env` 并添加你的 API keys：

   ```
   ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
   ANTHROPIC_API_KEY=sk-ant-api03-...
   ```

4. **安装依赖：**
   ```bash
   # With venv activated
   pip install -r requirements.txt
   ```

### 第 2 步：完成 Notebook

从 **[低延迟语音助手 Notebook](./low_latency_stt_claude_tts.ipynb)** 开始。这个交互式指南将教你：

- 如何使用 ElevenLabs 进行语音转文本
- 如何生成 Claude 响应并测量延迟
- 流式处理如何降低首个 token 的到达时间
- 如何流式进行文本转语音以实现更快的音频播放
- 不同流式方案之间的权衡
- 为什么 WebSocket 流式处理在延迟和质量之间提供了最佳平衡

该 notebook 在每一步都包含性能指标和对比，帮助你理解每项优化带来的影响。

### 第 3 步：试用生产脚本

在理解 notebook 中的概念之后，运行 **[WebSocket 流式脚本](./stream_voice_assistant_websocket.py)**，体验一个功能完整的语音助手：

```bash
python stream_voice_assistant_websocket.py
```

**它的工作方式：**
1. 按 Enter 开始录音
2. 对着麦克风说出你的问题
3. 按 Enter 停止录音
4. 助手会用自然语音进行回应
5. 重复上述过程，或按 Ctrl+C 退出

该脚本展示了以下可用于生产的实现方式：
- 使用 sounddevice 进行实时麦克风录音
- 带上下文保留的连续对话
- 基于 WebSocket 的流式处理以实现最小延迟
- 用于无缝播放的自定义音频队列

## 故障排查

### 音频爆音或噼啪声

**症状：** 播放过程中你可能偶尔会听到短暂的爆音、咔嗒声或音频中断。

**说明：**

这是因为脚本使用的是 MP3 格式音频，而 ElevenLabs 免费套餐要求使用这种格式。当以实时分块方式流式传输 MP3 数据时，FFmpeg 偶尔会收到无法解码的不完整帧。通常发生在以下情况：
- 流开始时（首个分块可能太小）
- 短暂网络延迟期间
- 音频生成结束时（最后一个分块可能不完整）

脚本会通过自动跳过这些失败分块来处理这种情况（在音频解码逻辑中使用 try-except 模式），从而避免在控制台中出现错误，但可能会导致短暂的音频空隙，表现为爆音或咔嗒声。

**影响：**
- 音频播放会继续正常进行
- 短暂爆音或咔嗒声通常难以察觉或影响很小
- WebSocket 连接会保持稳定
- 不会丢失任何功能

**解决方案：**

在免费套餐下使用 MP3 格式时，这是预期行为。如果你希望彻底消除音频爆音：
1. 升级到付费 ElevenLabs 套餐
2. 修改脚本，使用 `pcm_44100` 格式而非 MP3
3. PCM 格式可提供更干净的流式传输，不会有解码问题

### API Key 问题

**症状：** `AssertionError: ELEVENLABS_API_KEY is not set` 或 `AssertionError: ANTHROPIC_API_KEY is not set`

**解决方案：**
1. 确认你已将 `.env.example` 复制为 `.env`：`cp .env.example .env`
2. 编辑 `.env`，确认两个 API key 都已正确设置
3. 检查 API key 中是否有拼写错误或多余空格
4. 确认你的 ElevenLabs key 具备所需权限（见第 1 步）

### 依赖问题

**症状：** 出现 `ImportError: PortAudio library not found` 之类的错误，或音频播放失败

**解决方案：**

**macOS:**
```bash
brew install portaudio ffmpeg
```

**Ubuntu/Debian:**
```bash
sudo apt-get install portaudio19-dev ffmpeg
```

**Windows:**
- 从 [ffmpeg.org](https://ffmpeg.org/download.html) 安装 FFmpeg
- 将 FFmpeg 加入系统 PATH
- 在 Windows 上，PortAudio 通常会随着 sounddevice 自动安装

然后重新安装 Python 依赖：
```bash
pip install -r requirements.txt
```

### 麦克风权限

**症状：** `OSError: [Errno -9999] Unanticipated host error` 或无法访问麦克风

**解决方案：**
- **macOS:** 前往 System Preferences → Security &amp; Privacy → Privacy → Microphone，并启用 Terminal（或你的 Python IDE）
- **Windows:** 前往 Settings → Privacy → Microphone，并为 Python/Terminal 启用麦克风访问
- **Linux:** 检查你的用户是否在 `audio` 组中：`sudo usermod -a -G audio $USER`（然后注销并重新登录）

测试你的麦克风设置：

```bash
python -c "import sounddevice as sd; print(sd.query_devices())"
```

### WebSocket 连接失败

**症状：** 连接错误、超时或流中断

**解决方案：**
1. 检查你的网络连接是否稳定
2. 确认防火墙没有阻止 WebSocket 连接（端口 443）
3. 尝试暂时关闭 VPN 或代理
4. 确认你没有超出 API 速率限制（可在 ElevenLabs 控制台查看使用情况）

如果你仍然遇到问题，请查看 [ElevenLabs 状态页](https://status.elevenlabs.io/) 获取服务更新。

## 项目灵感

当你熟悉这个语音助手之后，以下是一些可供构建的启发性项目：

- **会议记录助手** - 实时录制并转写会议，然后使用 Claude 从对话中生成摘要、行动项和关键要点。

- **语言学习导师** - 用任意语言进行对话练习，并获得实时反馈。Claude 可以纠正发音、建议更好的表达方式，并根据你的水平调整难度。

- **交互式讲故事工具** - 创建“选择你自己的冒险”游戏，由 Claude 讲述故事，并根据你说出的选择进行回应，还可以为不同角色配上不同声音。

- **免手操作的编程助手** - 在双手保持键盘操作的同时，用语音描述代码变更、bug 或新功能。非常适合橡皮鸭调试或单人结对编程。

- **语音控制的智能家居** - 为家居设备控制构建自然对话界面。你可以提出“现在冷到该开暖气了吗？”这类复杂问题，而不只是简单的开/关命令。

- **个人语音日记** - 通过说出想法来记录每日日志。Claude 可以按主题整理条目、跟踪你的情绪变化，并在你需要时找出相关的过往记录。

## 关于 ElevenLabs 的更多信息

以下是一些有助于加深理解的资源：

- [ElevenLabs 平台](https://elevenlabs.io/) - 官方网站
- [API 文档](https://elevenlabs.io/docs/overview) - 完整 API 参考
- [语音库](https://elevenlabs.io/voice-library) - 浏览可用声音
- [API Playground](https://elevenlabs.io/app/speech-synthesis/text-to-speech) - 交互式测试声音
- [Python SDK](https://github.com/elevenlabs/elevenlabs-python) - 官方 Python SDK
