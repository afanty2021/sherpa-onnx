# Kotlin API 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / Kotlin API 示例

## 模块概述

本模块包含 Sherpa-ONNX Kotlin API 的使用示例，展示了如何使用 Kotlin 语言进行离线语音处理。这些示例主要运行在 JVM 上，通过 JNI 调用底层的 C++ 实现。

## 文件结构

```
kotlin-api-examples/
├── run.sh                   # 构建和运行脚本
├── test_version.kt          # 版本测试示例
├── VersionInfo.kt           # 版本信息获取
├── faked-asset-manager.kt   # 模拟资源管理器
├── faked-log.kt            # 模拟日志输出
└── [符号链接到 API 定义]    # 指向实际的 Kotlin API 定义
```

## 主要功能示例

### 1. 语音识别 (ASR)
- **在线识别**：实时流式语音识别
- **离线识别**：文件批量语音识别
- **多种模型支持**：Zipformer、Paraformer、Whisper、SenseVoice 等

### 2. 语音合成 (TTS)
- **多语言合成**：支持中文、英文、日文等
- **多说话人**：不同说话人风格
- **流式输出**：实时语音生成

### 3. 说话人处理
- **说话人识别**：识别音频中的说话人
- **说话人分离**：区分不同说话人片段
- **说话人日志**：带时间戳的标注

### 4. 其他功能
- **语音增强**：降噪处理
- **关键词检测**：特定词汇识别
- **音频标注**：音频事件检测
- **标点恢复**：自动添加标点符号

## 快速开始

### 1. 环境要求
- JDK 8 或更高版本
- Kotlin compiler
- CMake 3.16+
- C++ compiler

### 2. 构建 JNI 库
```bash
# 自动构建
./run.sh
```

### 3. 手动构建
```bash
# 创建构建目录
mkdir -p ../build
cd ../build

# 配置 CMake
cmake \
  -DSHERPA_ONNX_ENABLE_JNI=ON \
  -DSHERPA_ONNX_ENABLE_PYTHON=OFF \
  -DSHERPA_ONNX_ENABLE_TESTS=OFF \
  ..

# 编译
make -j4

# 返回示例目录
cd ../kotlin-api-examples
```

## 使用示例

### 版本信息测试

```kotlin
// test_version.kt
import com.k2fsa.sherpa.onnx.VersionInfo

fun main() {
    val version = VersionInfo.get()
    println("Sherpa-ONNX version: ${version.version}")
    println("Build date: ${version.buildDate}")
}
```

### 在线语音识别

```kotlin
import com.k2fsa.sherpa.onnx.*

fun main() {
    // 配置模型
    val modelConfig = sherpaOnnxOnlineModelConfig(
        tokens = "tokens.txt",
        transducer = sherpaOnnxOnlineTransducerModelConfig(
            encoder = "encoder.onnx",
            decoder = "decoder.onnx",
            joiner = "joiner.onnx"
        ),
        numThreads = 1,
        modelType = "zipformer"
    )

    // 配置识别器
    val recognizerConfig = sherpaOnnxOnlineRecognizerConfig(
        modelConfig = modelConfig,
        enableEndpoint = true
    )

    // 创建识别器
    val recognizer = OnlineRecognizer(recognizerConfig)

    // 创建音频流
    val stream = recognizer.createStream()

    // 模拟音频输入
    val audioData = FloatArray(1600) // 100ms @ 16kHz
    // ... 填充音频数据 ...

    // 识别
    stream.acceptWaveform(16000, audioData)
    recognizer.decode(stream)

    // 获取结果
    val result = recognizer.getResult(stream)
    println("识别结果: ${result.text}")

    // 清理
    recognizer.release()
}
```

### 语音合成

```kotlin
import com.k2fsa.sherpa.onnx.*

fun main() {
    // 配置 TTS 模型
    val modelConfig = sherpaOnnxOfflineTtsModelConfig(
        vits = sherpaOnnxOfflineTtsVitsModelConfig(
            model = "vits.onnx",
            lexicon = "lexicon.txt",
            tokens = "tokens.txt"
        ),
        numThreads = 1
    )

    // 创建模型
    val model = OfflineTtsModel(modelConfig)

    // 生成语音
    val audio = model.generate("Hello, world!", sid = 0)

    // 保存音频
    val outputFile = "output.wav"
    audio.save(outputFile)

    println("音频已保存到: $outputFile")
    println("采样率: ${audio.sampleRate}")
    println("时长: ${audio.duration} 秒")

    // 清理
    model.release()
}
```

### 说话人识别

```kotlin
import com.k2fsa.sherpa.onnx.*

fun main() {
    // 配置说话人模型
    val modelConfig = sherpaOnnxSpeakerEmbeddingExtractorModelConfig(
        model = "speaker_model.onnx",
        numThreads = 1
    )

    // 创建提取器
    val extractor = SpeakerEmbeddingExtractor(modelConfig)

    // 提取说话人特征
    val audioFile = "speaker.wav"
    val embedding1 = extractor.compute(audioFile)

    // 计算相似度
    val similarity = cosSimilarity(embedding1, embedding1)
    println("相似度: $similarity")

    // 清理
    extractor.release()
}
```

## 构建和运行

### 单个示例
```bash
# 编译
kotlinc-jvm -include-runtime -d example.jar \
  example.kt \
  VersionInfo.kt

# 运行
java -Djava.library.path=../build/lib -jar example.jar
```

### 使用构建脚本
```bash
# 运行所有测试
./run.sh

# 运行特定测试
./run.sh testVersion
./run.sh testSpeakerEmbeddingExtractor
```

## 模型下载

### 自动下载
运行示例时会自动下载所需模型：
```bash
# 运行脚本会自动下载模型到当前目录
./run.sh
```

### 手动下载
```bash
# 语音识别模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2

# 语音合成模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-zh-hf-pan-shuang-ep.tar.bz2

# 说话人识别模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/speaker-recognition-models/3dspeaker_speech_eres2net_large_sv_zh-cn_3dspeaker_16k.onnx
```

## 性能优化

### 1. 模型量化
使用 int8 量化模型：
```kotlin
val modelConfig = sherpaOnnxOnlineModelConfig(
    // ...
    modelType = "zipformer",
    modelingUnit = "cjkchar+latin"
)
```

### 2. 多线程配置
```kotlin
val modelConfig = sherpaOnnxOnlineModelConfig(
    // ...
    numThreads = Runtime.getRuntime().availableProcessors()
)
```

### 3. 内存管理
及时释放资源：
```kotlin
// 使用 try-with-resources 或手动释放
recognizer.release()
model.release()
stream.release()
```

## 常见问题

### 1. JNI 库找不到
确保设置了正确的库路径：
```bash
export LD_LIBRARY_PATH=$PWD/../build/lib:$LD_LIBRARY_PATH
```

### 2. 模型文件路径错误
使用绝对路径或确保模型在当前目录：
```kotlin
val encoder = "/path/to/encoder.onnx"  // 绝对路径
val decoder = "./decoder.onnx"          // 相对路径
```

### 3. 内存不足
减少线程数或使用量化模型：
```kotlin
val modelConfig = sherpaOnnxOnlineModelConfig(
    // ...
    numThreads = 1
)
```

## 相关文档

- [Kotlin API 文档](../sherpa-onnx/kotlin-api/CLAUDE.md)
- [Android 集成](../android/CLAUDE.md)
- [Java API 示例](../java-api-examples/CLAUDE.md)
- [Sherpa-ONNX Kotlin 文档](https://k2-fsa.github.io/sherpa/onnx/kotlin-api/)

---

*更多示例和详细信息请参考：https://github.com/k2-fsa/sherpa-onnx/tree/master/kotlin-api-examples*