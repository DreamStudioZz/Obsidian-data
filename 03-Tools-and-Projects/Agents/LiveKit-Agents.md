---
tags:
  - ai-tool
  - open-source
  - status/adopted
category: Agents
github: https://github.com/livekit/agents
stars: "10.8k+"
license: Apache-2.0
date_added: 2026-11-17
---

# 📦 LiveKit Agents

> **一句话简介**：全球领先的开源实时多模态与全双工语音智能体开发框架。依托 LiveKit 工业级 WebRTC 分布式流媒体底座，为开发者提供端到端延迟低至 300~500ms 的实时语音交互、低延迟视觉流理解、毫秒级智能打断（Interruption Handling）、端到端声音流直通（Native Audio-to-Audio）以及开箱即用的电话 SIP 网关接入能力。

---

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub: livekit/agents](https://github.com/livekit/agents) |
| **研发机构** | LiveKit 团队及全球开源社区 |
| **技术栈** | Python 3.10+, WebRTC, asyncio, PyTorch / ONNX, TypeScript / React, SIP |
| **许可证** | Apache-2.0 License |
| **核心机制** | 客户端/服务端全双工 WebRTC 音视频传输、Silero VAD（语音活动检测）+ 深度学习 Turn Detection 意图断句、流式管道级联（Streaming STT -> LLM -> TTS）、原生多模态直接对齐（OpenAI Realtime API / Gemini Multimodal Live API）、抢话即时音频缓冲熔断与任务取消机制、SIP 电话网络穿透 |
| **生态集成** | STT: Deepgram, Whisper, AssemblyAI; LLM: OpenAI, Anthropic, Groq, Ollama; TTS: ElevenLabs, Cartesia, PlayHT, Azure; SIP: Twilio, Telnyx |
| **相关关联** | 关联实践: [[实时全双工语音智能体架构：基于 LiveKit Agents 的低延迟 WebRTC 管道、智能打断与多模态会话状态机实战]], [[2026-11-17-AI-Digest]], [[FastMCP]], [[CopilotKit]], [[Browser-Use]], [[LangGraph]] |

---

## 🚀 核心特性与架构亮点

```mermaid
flowchart TD
    subgraph ClientUser [客户端用户与交互终端]
        UserMic["用户语音采集 (Microphone Audio Stream)"]
        UserCamera["用户实时摄像头/屏幕流 (Video Frames)"]
        UserSpeaker["实时音频播放 (Speaker Output)"]
    end

    subgraph LiveKitInfra [LiveKit 工业级 WebRTC 传输网络]
        SFUServer["LiveKit WebRTC SFU 服务器 (低延迟音视频网格, <50ms RTT)"]
        DataChannel["双向数据通道 Data Channel (控制信令 / 状态事件)"]
    end

    subgraph LiveKitAgentServer [LiveKit Agents 服务端工作节点 (Worker)]
        subgraph IngestionPipeline [1. 输入感知与断句]
            VADEngine["Silero VAD (能量与语音检测, 毫秒级触发)"]
            TurnDetector["Turn Detector 深度判定 (判断是短暂沉思停顿还是意图表达完毕)"]
            FrameSampler["视频帧自适应降采样与关键帧抽取"]
        end

        subgraph InterruptionEngine [2. 智能打断与状态机控制器]
            BargeInController["抢话截断控制器 (Barge-in Controller)"]
            FlushAudioBuffer["清空客户端播放缓冲 (Flush Playback Buffer)"]
            CancelPendingTasks["取消在途 LLM 生成与 TTS 流式任务"]
        end

        subgraph BrainAndSynthesis [3. 双模决策与多模态流式引擎]
            PipelineMode["级联模式: Streaming STT ➔ 流式 LLM ➔ 流式 TTS (Cartesia/Deepgram)"]
            DirectMode["原生端到端模式: OpenAI Realtime / Gemini Live 双向音频流"]
            ToolBus["Function Calling 业务工具执行 (动态查询 / 外部 API)"]
        end
    end

    UserMic ==> SFUServer
    UserCamera ==> SFUServer
    SFUServer ==> VADEngine & FrameSampler
    VADEngine ==> TurnDetector
    TurnDetector ==> BargeInController
    BargeInController -.->|用户抢话中断| FlushAudioBuffer & CancelPendingTasks
    TurnDetector ==> PipelineMode & DirectMode
    PipelineMode & DirectMode <--> ToolBus
    PipelineMode & DirectMode ==> SFUServer
    SFUServer ==> UserSpeaker
```

1. **破除 2~3 秒传统延迟枷锁：毫秒级全链路流式流水线**：
   - 传统的语音 AI 流程是“录音完毕 -> 整段上传转文字（STT）-> 提交大模型等待全部生成 -> 文本转语音合成（TTS）-> 播放”，交互延迟通常高达 2.5 到 4 秒，极具机械割裂感。
   - LiveKit Agents 将整个链路全面打散为毫秒级流水线（Pipelining）：用户说话的第一秒起，音频以 20ms 的 PCM 包形式通过 WebRTC 直流进服务内存；STT 流式实时出词，LLM 首字吐出即刻送入超低延迟流式 TTS 引擎（如 Cartesia 或 ElevenLabs WebSocket），端到端从用户停顿到听到第一声回答的延迟被极限压缩至 300~500ms 以内，达到人类自然对话响应水准。
2. **极速智能抢话打断（Barge-In Handling）与状态机熔断**：
   - 真实人机对话中，用户随时可能打断 Agent 的长篇大论。如果系统缺乏打断机制，Agent 会在用户说话时继续自说自话，产生严重的“抢麦”混乱。
   - LiveKit Agents 内置了严密的状态机。一旦 Silero VAD 判定用户产生新发音，系统在 10 毫秒内向 WebRTC 数据通道发送 `truncate` 信令清空客户端音频播放队列，同时立即向底层 `asyncio` 发出任务取消信号，掐断正在推理的 LLM 与 TTS 任务，Agent 瞬间收声并无缝转入倾听状态。
3. **原生全双工（Native Audio-to-Audio）与传统级联双轨支持**：
   - 既全面支持传统的“STT + LLM + TTS”解耦组合（支持灵活替换任何私有部署的模型，如 Whisper + vLLM + MeloTTS），又原生集成了以 OpenAI Realtime API 和 Google Gemini Multimodal Live API 为代表的原生音频直通多模态大模型，支持直接理解语调、笑声、停顿等副语言特征。
4. **企业级电话集成（SIP Telephony）开箱即用**：
   - 原生支持 SIP（Session Initiation Protocol）协议。可通过 Twilio、Telnyx 或私有 SIP 中继，一键将 AI 智能体挂接到真实电话号码网络，直接充当银行外呼回访、在线客服、紧急挂号等传统电信业务的实时接线员。
5. **多模态视觉流（Vision-Language Streaming）感知**：
   - Agent 不仅能“听”，还能实时“看”。客户端可通过 WebRTC 将手机摄像头或屏幕录制流推送至房间，Agent 服务端能够根据对话节奏智能抽取高分辨率关键帧，实时理解用户当前屏幕或现实环境中的物理场景并展开讲解。

---

## 🛠️ 快速上手与配置

### 1. 环境依赖与安装

```bash
# 安装 LiveKit Agents 核心库及其典型插件 (Silero VAD, Deepgram STT, OpenAI LLM, Cartesia TTS)
pip install "livekit-agents[all]" \
            livekit-plugins-silero \
            livekit-plugins-deepgram \
            livekit-plugins-openai \
            livekit-plugins-cartesia
```

### 2. 极速构建全双工语音助手示例

创建 `agent.py` 文件，展示一个标准的具备 VAD 检测、智能打断与工具调用的语音助理：

```python
import asyncio
from livekit.agents import (
    AutoSubscribe,
    JobContext,
    WorkerOptions,
    cli,
    llm,
)
from livekit.agents.voice_assistant import VoiceAssistant
from livekit.plugins import cartesia, deepgram, openai, silero


async def entrypoint(ctx: JobContext):
    # 自动连接到当前分配的 WebRTC 房间并订阅音视频流
    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)

    # 初始化上下文意图与系统提示词
    initial_ctx = llm.ChatContext().append(
        role="system",
        text="你是一个充满活力、专业幽默的个人 AI 语音助理。你的回答应该口语化、精练，且避免长篇大论。",
    )

    # 装配超低延迟流式语音全链路
    assistant = VoiceAssistant(
        vad=silero.VAD.load(),                     # 本地轻量级 Silero VAD 语音活动检测
        stt=deepgram.STT(),                         # Deepgram Nova-2 极速流式语音识别
        llm=openai.LLM(model="gpt-4o-mini"),        # 高响应速度 LLM
        tts=cartesia.TTS(voice="248be419-c632-4f23-add1-073ca7cd4236"), # Cartesia Sonic 80ms 极速语音合成
        chat_ctx=initial_ctx,
        interruption_speech_duration=0.4,           # 用户持续发声 0.4 秒即判定为明确抢话打断
    )

    # 注册一个可供语音实时调用的业务工具 (Function Calling)
    @assistant.function_context.register
    def query_server_status(service_name: str) -> str:
        """查询指定后台微服务的健康状态"""
        return f"微服务 {service_name} 当前运行平稳，CPU 负载 12%，无报警事件。"

    # 启动语音助理并使其在房间中驻留
    assistant.start(ctx.room)

    # 进房主动问候用户
    await assistant.say("你好！我是你的实时智能体助手，有什么我可以帮你的吗？", allow_interruptions=True)


if __name__ == "__main__":
    # 使用 LiveKit 官方 CLI 启动多进程 Worker 集群
    cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint))
```

### 3. 启动并测试调试

```bash
# 设置 LiveKit Cloud 或本地自建 LiveKit 服务的环境变量
export LIVEKIT_URL="wss://your-project.livekit.cloud"
export LIVEKIT_API_KEY="your-api-key"
export LIVEKIT_API_SECRET="your-api-secret"
export OPENAI_API_KEY="sk-..."
export DEEPGRAM_API_KEY="..."
export CARTESIA_API_KEY="..."

# 启动本地开发模式 (支持热重载与控制台可视化事件瀑布)
python agent.py dev
```

---

## 💡 工程实战点评与适用场景

- **推荐使用场景**：
  - **新一代实时全双工语音助手**：替代传统按键式录音问答，提供极具沉浸感、低至 400ms 响应的面对面级自然语音交谈体验。
  - **电信级自动化语音客服与外呼系统**：通过 SIP 协议直连运营商线路，处理高并发呼叫中心咨询、回访与订单确认。
  - **多模态具身视听陪伴与教学演练**：结合视频流分析，在英语口语教练、模拟面试、车载中控、远程代码 Pair-programming 等场景中同时观察屏幕并实时语音指引。
- **潜在不足 / 局限性**：
  - **网络抖动与 NAT 穿透敏感**：WebRTC 依赖良好的 UDP 网络环境。在跨国或极其严苛的企业防火墙限制下，需要合理配置 TURN/STUN 服务器以防丢包或回落为高延迟 TCP。
  - **高并发下第三方 API 成本与配额压力**：全双工场景下连续流式请求会持续占用 STT/TTS 会话通道，在大规模部署时需要做好连接池复用与用户静默检测节流。
- **个人评估结论**：**极度推荐采用（Adopted）**。LiveKit Agents 彻底补全了智能体技术栈在“物理音频流与亚秒级实时交互”维度的空白，是当下构建工业级全双工语音与多模态 Agent 的事实标准基座。
