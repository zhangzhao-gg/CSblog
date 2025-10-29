Qwen-Omni模型当前功能是能够支持多模态输入输出

调研结果，数据来源于论文，魔塔社区反馈，官方写的介绍信息，huggingFace的demo测试

开源的三个模型：
Qwen/Qwen3-Omni-30B-A3B-Captioner
Qwen/Qwen3-Omni-30B-A3B-Instruct
Qwen/Qwen3-Omni-30B-A3B-Thinking

**音频相关能力：**
其中Qwen3-Omni-30B-A3B-Instruct模型根据魔塔介绍支持：实时音频/视频交互，低延迟流媒体，具有自然的轮流发言和即时文本或语音响应。

**测试方法：**
若本地测试需要使用 transformer包
VLLM直接下载的版本暂不支持音频输出，若需使用VLLM部署，需要从最新版源码编译实现。
从社区反馈来看，音频输出速度比较慢。根据论文来看，流式输出的每一包是80ms

**是否智能截断并且输出：**
大模型无法进行vad,他无法判断何时结束，所以vad需要我们来做。【目前存疑，因为若大模型无法知道何时结束，那么他是如何决定在什么时候输出的呢】

官方闭源模型：
qwen3-omni-flash-realtime 支持音频输入输出，一定支持实时音频流式传输
qwen3-omni-flash-不确定能不能实时没说
