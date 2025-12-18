# Swift API 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / Swift API 示例

## 模块概述

本模块包含 Sherpa-ONNX Swift API 的使用示例，展示了如何在 Swift 应用中实现各种语音处理功能。这些示例涵盖了语音识别、语音合成、说话人识别、关键词检测等完整的功能集。

## 功能分类

### 🎤 语音识别 (ASR) 示例

#### 流式识别
- **decode-file.swift**: 基本的文件语音识别
- **decode-file-t-one-streaming.swift**: T-One 俄语流式识别
- **decode-file-sense-voice-with-hr.swift**: SenseVoice 带热词识别

#### 非流式识别
- **decode-file-non-streaming.swift**: 批量文件识别
- **dolphin-ctc-asr.swift**: Dolphin CTT 模型示例
- **fire-red-asr.swift**: FireRed 中文 ASR 模型
- **omnilingual-asr-ctc.swift**: Omnilingual 多语言 CTC

#### 高级功能
- **generate-subtitles.swift**: 自动生成字幕文件

### 🔊 语音合成 (TTS) 示例
- **vits-tts.swift**: VITS 文本转语音
- **matcha-tts.swift**: Matcha-TTS 快速合成
- **piper-tts.swift**: Piper 本地化 TTS

### 👥 说话人处理示例
- **compute-speaker-embeddings.swift**: 说话人特征提取
- **speaker-diarization.swift**: 说话人分离
- **speaker-verification.swift**: 说话人验证

### 🎯 其他功能示例

#### 关键词检测
- **keyword-spotting-from-file.swift**: 从文件检测关键词

#### 标点恢复
- **add-punctuations.swift**: 离线标点恢复
- **add-punctuation-online.swift**: 在线标点恢复

#### 语音增强
- **speech-enhancement-gtcrn.swift**: GTCRN 语音增强
- **speech-enhancement.mdns.md**: 简单降噪

#### 音频处理
- **audio-tagging.swift**: 音频事件检测
- **vad-asr.swift**: 语音活动检测 + 识别

## 快速开始

### 1. 环境要求
- Swift 5.5+
- macOS 11.0+ / Linux / iOS 13.0+
- ONNX Runtime 库

### 2. 安装依赖
```bash
# macOS
brew install swift

# Linux (Ubuntu)
sudo apt-get install swift swift-lang

# 或使用 Swift 官方工具链
```

### 3. 编译和运行
```bash
# 编译单个示例
swiftc -I ./path/to/include -L ./path/to/lib example.swift -l sherpa-onnx

# 使用构建脚本
./run-decode-file.sh
```

## 使用示例

### 基本语音识别

```swift
// decode-file.swift
import Foundation
import SherpaOnnx

// 配置模型
let modelConfig = SherpaOnnxOnlineModelConfig(
  tokens: "tokens.txt",
  transducer: SherpaOnnxOnlineTransducerModelConfig(
    encoder: "encoder.onnx",
    decoder: "decoder.onnx",
    joiner: "joiner.onnx"
  ),
  numThreads: 1,
  modelType: "zipformer"
)

// 配置识别器
let recognizerConfig = SherpaOnnxOnlineRecognizerConfig(
  modelConfig: modelConfig,
  enableEndpoint: true,
  rule1MinSilenceDuration: 0.5
)

// 创建识别器
let recognizer = SherpaOnnxOnlineRecognizer(config: recognizerConfig)

// 创建流
let stream = recognizer.createStream()

// 读取音频文件
let audioFile = "test.wav"
let samples = readAudioFile(audioFile)

// 识别
stream.acceptWaveform(samples: samples)
recognizer.decode(stream)

// 获取结果
let result = recognizer.getResult(stream: stream)
print("识别结果: \(result.text)")
```

### 语音合成

```swift
// vits-tts.swift
import Foundation
import SherpaOnnx

// 配置 TTS 模型
let modelConfig = SherpaOnnxOfflineTtsModelConfig(
  vits: SherpaOnnxOfflineTtsVitsModelConfig(
    model: "vits.onnx",
    lexicon: "lexicon.txt",
    tokens: "tokens.txt"
  ),
  numThreads: 1
)

// 创建模型
let model = SherpaOnnxOfflineTtsModel(config: modelConfig)

// 生成语音
let audio = model.generate(text: "Hello, world!", sid: 0)

// 保存音频
let outputPath = "output.wav"
audio.save(filename: outputPath)

print("音频已保存到: \(outputPath)")
print("采样率: \(audio.sampleRate) Hz")
print("帧数: \(audio.numFrames)")
```

### 说话人识别

```swift
// compute-speaker-embeddings.swift
import Foundation
import SherpaOnnx

// 配置说话人模型
let modelConfig = SherpaOnnxSpeakerEmbeddingExtractorModelConfig(
  model: "speaker_model.onnx",
  numThreads: 1
)

// 创建提取器
let extractor = SherpaOnnxSpeakerEmbeddingExtractor(config: modelConfig)

// 提取特征
let audioFile1 = "speaker1.wav"
let audioFile2 = "speaker2.wav"

let embedding1 = extractor.compute(filename: audioFile1)
let embedding2 = extractor.compute(filename: audioFile2)

// 计算相似度
let similarity = sherpaOnnxComputeCosineSimilarity(
  x: embedding1,
  y: embedding2
)

print("说话人相似度: \(similarity)")
```

### 关键词检测

```swift
// keyword-spotting-from-file.swift
import Foundation
import SherpaOnnx

// 配置关键词检测
let modelConfig = SherpaOnnxKeywordSpotterModelConfig(
  transducer: SherpaOnnxOnlineTransducerModelConfig(
    encoder: "encoder.onnx",
    decoder: "decoder.onnx",
    joiner: "joiner.onnx"
  ),
  tokens: "tokens.txt"
)

// 配置关键词
let keywordsFile = "keywords.txt"  // 每行一个关键词
let config = SherpaOnnxKeywordSpotterConfig(
  modelConfig: modelConfig,
  keywordsFile: keywordsFile,
  keywordsThreshold: 0.5
)

// 创建检测器
let spotter = SherpaOnnxKeywordSpotter(config: config)

// 创建流
let stream = spotter.createStream()

// 处理音频
let audioFile = "test.wav"
let samples = readAudioFile(audioFile)

// 检测
stream.acceptWaveform(samples: samples)
spotter.decode(stream)

// 获取结果
let result = spotter.getResult(stream: stream)
if result.keyword.count > 0 {
  print("检测到关键词: \(result.keyword)")
  print("置信度: \(result.prob)")
}
```

## 运行脚本

每个示例都有对应的运行脚本：

### ASR 示例
```bash
# 基本识别
./run-decode-file.sh

# 非流式识别
./run-decode-file-non-streaming.sh

# SenseVoice 识别
./run-decode-file-sense-voice-with-hr.sh
```

### TTS 示例
```bash
# VITS TTS
./run-vits-tts.sh

# Matcha-TTS
./run-matcha-tts.sh
```

### 说话人处理
```bash
# 特征提取
./run-compute-speaker-embeddings.sh

# 说话人分离
./run-speaker-diarization.sh
```

## 模型下载

运行脚本会自动下载所需模型。手动下载：

```bash
# ASR 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2

# TTS 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-zh-hf-pan-shuang-ep.tar.bz2

# 说话人模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/speaker-recongition-models/3dspeaker_speech_eres2net_large_sv_zh-cn_3dspeaker_16k.onnx
```

## 性能优化

### 1. 多线程配置
```swift
let modelConfig = SherpaOnnxOnlineModelConfig(
  // ...
  numThreads: ProcessInfo.processInfo.processorCount
)
```

### 2. 模型量化
使用 int8 量化模型：
```swift
let modelConfig = SherpaOnnxOnlineModelConfig(
  // ...
  modelType: "zipformer2"
)
```

### 3. 内存管理
使用 `defer` 确保资源释放：
```swift
let recognizer = SherpaOnnxOnlineRecognizer(config: config)
defer {
  recognizer.release()
}
```

## 平台支持

### macOS
- 完全支持，包括 Metal 加速
- 使用 Homebrew 安装依赖

### Linux
- 支持 Ubuntu 18.04+
- 需要手动安装依赖库

### iOS
- 通过 iOS 模块集成
- 参见 [iOS SwiftUI 集成](../ios-swiftui/CLAUDE.md)

## 常见问题

### 1. 找不到库
确保设置了正确的库路径：
```bash
export DYLD_LIBRARY_PATH=/path/to/lib:$DYLD_LIBRARY_PATH
```

### 2. 模型加载失败
检查模型文件路径和格式：
```swift
guard FileManager.default.fileExists(atPath: modelPath) else {
  fatalError("模型文件不存在: \(modelPath)")
}
```

### 3. 编译错误
确保 Swift 版本兼容：
```bash
swift --version  # 应该是 5.5 或更高
```

## 相关文档

- [Swift API 文档](https://k2-fsa.github.io/sherpa/onnx/swift-api/)
- [iOS Swift 集成](../ios-swift/CLAUDE.md)
- [iOS SwiftUI 集成](../ios-swiftui/CLAUDE.md)
- [Swift Package Manager 集成](../ios/CLAUDE.md)

---

*更多示例和详细信息请参考：https://github.com/k2-fsa/sherpa-onnx/tree/master/swift-api-examples*