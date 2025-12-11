# C++ API 示例

<cite>
**本文档中引用的文件**  
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)
- [whisper-cxx-api.cc](file://cxx-api-examples/whisper-cxx-api.cc)
- [audio-tagging-zipformer-cxx-api.cc](file://cxx-api-examples/audio-tagging-zipformer-cxx-api.cc)
- [kws-cxx-api.cc](file://cxx-api-examples/kws-cxx-api.cc)
- [speech-enhancement-gtcrn-cxx-api.cc](file://cxx-api-examples/speech-enhancement-gtcrn-cxx-api.cc)
- [matcha-tts-en-cxx-api.cc](file://cxx-api-examples/matcha-tts-en-cxx-api.cc)
- [vad-cxx-api.cc](file://cxx-api-examples/vad-cxx-api.cc)
- [offline-punctuation-cxx-api.cc](file://cxx-api-examples/offline-punctuation-cxx-api.cc)
- [online-punctuation-cxx-api.cc](file://cxx-api-examples/online-punctuation-cxx-api.cc)
- [parakeet-tdt-simulate-streaming-microphone-cxx-api.cc](file://cxx-api-examples/parakeet-tdt-simulate-streaming-microphone-cxx-api.cc)
- [CMakeLists.txt](file://cxx-api-examples/CMakeLists.txt)
</cite>

## 目录
1. [项目结构](#项目结构)
2. [非流式语音识别](#非流式语音识别)
3. [流式语音识别](#流式语音识别)
4. [音频标签](#音频标签)
5. [语音合成](#语音合成)
6. [语音增强](#语音增强)
7. [标点符号添加](#标点符号添加)
8. [关键词检测](#关键词检测)
9. [语音活动检测](#语音活动检测)
10. [CMake构建系统](#cmake构建系统)
11. [运行示例的完整步骤](#运行示例的完整步骤)
12. [常见错误及解决方案](#常见错误及解决方案)

## 项目结构

C++ API示例代码位于`cxx-api-examples`目录中，包含了各种语音处理功能的示例实现。这些示例展示了如何使用sherpa-onnx的C++ API进行语音识别、语音合成、音频标签等任务。

```mermaid
graph TD
A[cxx-api-examples] --> B[语音识别]
A --> C[语音合成]
A --> D[音频标签]
A --> E[语音增强]
A --> F[标点符号添加]
A --> G[关键词检测]
A --> H[语音活动检测]
B --> B1[非流式识别]
B --> B2[流式识别]
C --> C1[离线TTS]
D --> D1[音频标签]
E --> E1[语音增强]
F --> F1[离线标点]
F --> F2[在线标点]
G --> G1[关键词检测]
H --> H1[语音活动检测]
```

**Diagram sources**
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)
- [whisper-cxx-api.cc](file://cxx-api-examples/whisper-cxx-api.cc)
- [audio-tagging-zipformer-cxx-api.cc](file://cxx-api-examples/audio-tagging-zipformer-cxx-api.cc)

**Section sources**
- [cxx-api-examples](file://cxx-api-examples)

## 非流式语音识别

非流式语音识别示例展示了如何使用完整的音频文件进行语音识别。以Whisper模型为例，通过`OfflineRecognizer`类实现非流式识别。

```mermaid
sequenceDiagram
participant 用户 as 用户
participant 配置 as 配置
participant 识别器 as 识别器
participant 流 as 流
participant 结果 as 结果
用户->>配置 : 创建OfflineRecognizerConfig
配置->>配置 : 设置模型路径和参数
配置->>识别器 : OfflineRecognizer : : Create(config)
识别器->>识别器 : 加载模型
识别器->>用户 : 返回识别器对象
用户->>流 : recognizer.CreateStream()
用户->>流 : stream.AcceptWaveform(音频数据)
用户->>识别器 : recognizer.Decode(&stream)
识别器->>结果 : recognizer.GetResult(&stream)
结果->>用户 : 返回识别结果
```

**Diagram sources**
- [whisper-cxx-api.cc](file://cxx-api-examples/whisper-cxx-api.cc)

**Section sources**
- [whisper-cxx-api.cc](file://cxx-api-examples/whisper-cxx-api.cc)

## 流式语音识别

流式语音识别示例展示了如何处理实时音频流。以Zipformer模型为例，通过`OnlineRecognizer`类实现流式识别，支持实时语音识别。

```mermaid
sequenceDiagram
participant 用户 as 用户
participant 配置 as 配置
participant 识别器 as 识别器
participant 流 as 流
participant 结果 as 结果
用户->>配置 : 创建OnlineRecognizerConfig
配置->>配置 : 设置模型路径和参数
配置->>识别器 : OnlineRecognizer : : Create(config)
识别器->>识别器 : 加载模型
识别器->>用户 : 返回识别器对象
用户->>流 : recognizer.CreateStream()
loop 处理音频流
用户->>流 : stream.AcceptWaveform(音频块)
识别器->>识别器 : IsReady(&stream)
识别器->>识别器 : Decode(&stream)
end
用户->>流 : stream.InputFinished()
识别器->>结果 : recognizer.GetResult(&stream)
结果->>用户 : 返回最终识别结果
```

**Diagram sources**
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)

**Section sources**
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)

## 音频标签

音频标签示例展示了如何使用Zipformer模型进行音频分类和标签识别。通过`AudioTagging`类实现音频事件检测。

```mermaid
sequenceDiagram
participant 用户 as 用户
participant 配置 as 配置
participant 标签器 as 标签器
participant 流 as 流
participant 事件 as 事件
用户->>配置 : 创建AudioTaggingConfig
配置->>配置 : 设置模型路径和参数
配置->>标签器 : AudioTagging : : Create(config)
标签器->>标签器 : 加载模型
标签器->>用户 : 返回标签器对象
用户->>流 : tagger.CreateStream()
用户->>流 : stream.AcceptWaveform(音频数据)
用户->>事件 : tagger.Compute(&stream)
事件->>用户 : 返回音频事件列表
```

**Diagram sources**
- [audio-tagging-zipformer-cxx-api.cc](file://cxx-api-examples/audio-tagging-zipformer-cxx-api.cc)

**Section sources**
- [audio-tagging-zipformer-cxx-api.cc](file://cxx-api-examples/audio-tagging-zipformer-cxx-api.cc)

## 语音合成

语音合成示例展示了如何使用MatchaTTS模型将文本转换为语音。通过`OfflineTts`类实现离线语音合成。

```mermaid
sequenceDiagram
participant 用户 as 用户
participant 配置 as 配置
participant TTS as TTS
participant 音频 as 音频
用户->>配置 : 创建OfflineTtsConfig
配置->>配置 : 设置模型路径和参数
配置->>TTS : OfflineTts : : Create(config)
TTS->>TTS : 加载模型
TTS->>用户 : 返回TTS对象
用户->>TTS : tts.Generate(文本, sid, speed)
alt 使用进度回调
TTS->>用户 : 调用ProgressCallback
用户->>TTS : 返回1继续生成
end
TTS->>音频 : 生成音频数据
音频->>用户 : 返回GeneratedAudio
用户->>文件 : WriteWave(输出文件, 音频)
```

**Diagram sources**
- [matcha-tts-en-cxx-api.cc](file://cxx-api-examples/matcha-tts-en-cxx-api.cc)

**Section sources**
- [matcha-tts-en-cxx-api.cc](file://cxx-api-examples/matcha-tts-en-cxx-api.cc)

## 语音增强

语音增强示例展示了如何使用GTCRN模型去除音频中的噪声。通过`OfflineSpeechDenoiser`类实现语音去噪。

```mermaid
sequenceDiagram
participant 用户 as 用户
participant 配置 as 配置
participant 去噪器 as 去噪器
participant 增强音频 as 增强音频
用户->>配置 : 创建OfflineSpeechDenoiserConfig
配置->>配置 : 设置模型路径
配置->>去噪器 : OfflineSpeechDenoiser : : Create(config)
去噪器->>去噪器 : 加载模型
去噪器->>用户 : 返回去噪器对象
用户->>去噪器 : sd.Run(音频数据, 大小, 采样率)
去噪器->>增强音频 : 执行去噪处理
增强音频->>用户 : 返回去噪后的音频
用户->>文件 : WriteWave(输出文件, 增强音频)
```

**Diagram sources**
- [speech-enhancement-gtcrn-cxx-api.cc](file://cxx-api-examples/speech-enhancement-gtcrn-cxx-api.cc)

**Section sources**
- [speech-enhancement-gtcrn-cxx-api.cc](file://cxx-api-examples/speech-enhancement-gtcrn-cxx-api.cc)

## 标点符号添加

标点符号添加示例展示了如何为无标点的文本添加适当的标点符号。包含离线和在线两种模式。

```mermaid
flowchart TD
Start([开始]) --> 配置["创建标点配置"]
配置 --> 加载["创建标点对象"]
加载 --> 检查["检查对象有效性"]
检查 --> |无效| 错误["输出错误信息"]
检查 --> |有效| 处理["调用AddPunctuation"]
处理 --> 输出["输出带标点的文本"]
输出 --> End([结束])
style Start fill:#f9f,stroke:#333
style End fill:#f9f,stroke:#333
```

**Diagram sources**
- [offline-punctuation-cxx-api.cc](file://cxx-api-examples/offline-punctuation-cxx-api.cc)
- [online-punctuation-cxx-api.cc](file://cxx-api-examples/online-punctuation-cxx-api.cc)

**Section sources**
- [offline-punctuation-cxx-api.cc](file://cxx-api-examples/offline-punctuation-cxx-api.cc)
- [online-punctuation-cxx-api.cc](file://cxx-api-examples/online-punctuation-cxx-api.cc)

## 关键词检测

关键词检测示例展示了如何在音频流中检测预定义的关键词。通过`KeywordSpotter`类实现关键词检测功能。

```mermaid
sequenceDiagram
participant 用户 as 用户
participant 配置 as 配置
participant KWS as KWS
participant 流 as 流
participant 结果 as 结果
用户->>配置 : 创建KeywordSpotterConfig
配置->>配置 : 设置模型路径和关键词文件
配置->>KWS : KeywordSpotter : : Create(config)
KWS->>KWS : 加载模型
KWS->>用户 : 返回KWS对象
用户->>流 : kws.CreateStream()
用户->>流 : stream.AcceptWaveform(音频数据)
用户->>流 : stream.InputFinished()
loop 检测关键词
KWS->>KWS : IsReady(&stream)
KWS->>KWS : Decode(&stream)
KWS->>结果 : GetResult(&stream)
结果->>用户 : 返回检测到的关键词
用户->>KWS : kws.Reset(&stream)
end
```

**Diagram sources**
- [kws-cxx-api.cc](file://cxx-api-examples/kws-cxx-api.cc)

**Section sources**
- [kws-cxx-api.cc](file://cxx-api-examples/kws-cxx-api.cc)

## 语音活动检测

语音活动检测示例展示了如何检测音频中的语音活动并去除静音部分。支持Silero-VAD和Ten-VAD两种模型。

```mermaid
flowchart TD
Start([开始]) --> 配置["创建VadModelConfig"]
配置 --> 选择["选择VAD模型"]
选择 --> |Silero-VAD| 设置1["设置silero_vad参数"]
选择 --> |Ten-VAD| 设置2["设置ten_vad参数"]
设置1 --> 创建["创建VoiceActivityDetector"]
设置2 --> 创建
创建 --> 检查["检查VAD有效性"]
检查 --> |无效| 错误["输出错误信息"]
检查 --> |有效| 处理["处理音频流"]
处理 --> 分块["分块处理音频"]
分块 --> 接受["vad.AcceptWaveform"]
接受 --> 判断["判断是否为EOF"]
判断 --> |否| 循环["继续处理"]
判断 --> |是| 刷新["vad.Flush()"]
循环 --> 非空["vad.IsEmpty()"]
刷新 --> 非空
非空 --> |否| 结束["结束"]
非空 --> |是| 取出["vad.Front()"]
取出 --> 输出["输出时间段"]
输出 --> 保存["保存非静音音频"]
保存 --> 结束
```

**Diagram sources**
- [vad-cxx-api.cc](file://cxx-api-examples/vad-cxx-api.cc)

**Section sources**
- [vad-cxx-api.cc](file://cxx-api-examples/vad-cxx-api.cc)

## CMake构建系统

C++ API示例使用CMake作为构建系统，`cxx-api-examples/CMakeLists.txt`文件定义了所有示例的构建规则。

```mermaid
graph TD
A[CMakeLists.txt] --> B[添加可执行文件]
A --> C[链接库]
A --> D[条件编译]
B --> B1[streaming-zipformer-cxx-api]
B --> B2[whisper-cxx-api]
B --> B3[audio-tagging-ced-cxx-api]
B --> B4[kws-cxx-api]
B --> B5[speech-enhancement-gtcrn-cxx-api]
C --> C1[链接sherpa-onnx-cxx-api]
C --> C2[条件链接portaudio_static]
C --> C3[条件链接asound]
D --> D1[SHERPA_ONNX_ENABLE_PORTAUDIO]
D --> D2[SHERPA_ONNX_HAS_ALSA]
D --> D3[SHERPA_ONNX_ENABLE_TTS]
```

**Diagram sources**
- [CMakeLists.txt](file://cxx-api-examples/CMakeLists.txt)

**Section sources**
- [CMakeLists.txt](file://cxx-api-examples/CMakeLists.txt)

## 运行示例的完整步骤

运行C++ API示例的完整步骤包括环境准备、模型下载、构建和执行。

```mermaid
flowchart TD
A[准备环境] --> B[安装依赖]
B --> C[克隆仓库]
C --> D[创建构建目录]
D --> E[配置CMake]
E --> F[编译项目]
F --> G[下载模型]
G --> H[运行示例]
A --> A1[安装CMake]
A --> A2[安装编译器]
A --> A3[安装PortAudio]
E --> E1[指定安装路径]
E --> E2[启用功能开关]
G --> G1[使用wget下载]
G --> G2[解压模型文件]
H --> H1[设置环境变量]
H --> H2[执行可执行文件]
```

**Section sources**
- [CMakeLists.txt](file://cxx-api-examples/CMakeLists.txt)
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)

## 常见错误及解决方案

以下是运行C++ API示例时可能遇到的常见错误及其解决方案。

```mermaid
graph TD
A[常见错误] --> B[模型加载失败]
A --> C[音频文件读取失败]
A --> D[PortAudio初始化失败]
A --> E[内存不足]
A --> F[线程数设置不当]
B --> B1[检查模型路径]
B --> B2[验证文件完整性]
B --> B3[确认模型兼容性]
C --> C1[检查文件路径]
C --> C2[验证文件格式]
C --> C3[确认读取权限]
D --> D1[检查设备索引]
D --> D2[验证PortAudio安装]
D --> D3[设置环境变量]
E --> E1[减少线程数]
E --> E2[优化内存使用]
E --> E3[分批处理数据]
F --> F1[调整num_threads]
F --> F2[监控CPU使用率]
F --> F3[平衡性能与资源]
```

**Section sources**
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)
- [whisper-cxx-api.cc](file://cxx-api-examples/whisper-cxx-api.cc)
- [parakeet-tdt-simulate-streaming-microphone-cxx-api.cc](file://cxx-api-examples/parakeet-tdt-simulate-streaming-microphone-cxx-api.cc)