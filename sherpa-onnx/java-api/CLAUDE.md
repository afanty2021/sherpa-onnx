[根目录](../../../CLAUDE.md) > [sherpa-onnx](../../CLAUDE.md) > **java-api**

# Sherpa-ONNX Java API

> 更新时间：2025-12-18 10:45:53

## 模块职责

Java API 模块为 Sherpa-ONNX 提供了完整的 Java 语言绑定，使得 Java 和 Kotlin 应用能够方便地使用所有语音处理功能。该模块通过 JNI（Java Native Interface）调用底层的 C++ 实现，提供了面向对象的 Java API。

## 入口与启动

### 主要入口类
- **包路径**: `com.k2fsa.sherpa.onnx`
- **核心类**:
  - `OfflineRecognizer` - 离线语音识别
  - `OnlineRecognizer` - 在线语音识别
  - `OfflineTts` - 离线语音合成
  - `Vad` - 语音活动检测
  - `SpeakerEmbeddingExtractor` - 说话人嵌入提取

### 构建配置
- **Maven 坐标**: `com.litongjava:sherpa-onnx-java-api:1.0.1`
- **构建文件**: `pom.xml`
- **Makefile**: 用于本地构建

## 对外接口

### 1. 语音识别 (ASR)
```java
// 离线识别
OfflineRecognizerConfig config = new OfflineRecognizerConfig(...);
OfflineRecognizer recognizer = new OfflineRecognizer(config);
OfflineRecognizerResult result = recognizer.recognize(audioFile);

// 在线识别
OnlineRecognizerConfig config = new OnlineRecognizerConfig(...);
OnlineRecognizer recognizer = new OnlineRecognizer(config);
OnlineStream stream = recognizer.createStream();
stream.acceptWaveform(samples, sampleRate);
OnlineRecognizerResult result = recognizer.decode(stream);
```

### 2. 语音合成 (TTS)
```java
OfflineTtsConfig config = new OfflineTtsConfig(...);
OfflineTts tts = new OfflineTts(config);
GeneratedAudio audio = tts.generate("Hello, world!");
WaveWriter.write("output.wav", audio.getSamples(), audio.getSampleRate());
```

### 3. 说话人处理
```java
// 说话人嵌入提取
SpeakerEmbeddingExtractorConfig config = new SpeakerEmbeddingExtractorConfig(...);
SpeakerEmbeddingExtractor extractor = new SpeakerEmbeddingExtractor(config);
float[] embedding = extractor.extract(audioFile);

// 说话人日志
OfflineSpeakerDiarizationConfig config = new OfflineSpeakerDiarizationConfig(...);
OfflineSpeakerDiarization diarization = new OfflineSpeakerDiarization(config);
List<OfflineSpeakerDiarizationSegment> segments = diarization.process(audioFile);
```

### 4. 其他功能
- **VAD**: 语音活动检测
- **关键词检测**: `KeywordSpotter`
- **语音增强**: `OfflineSpeechDenoiser`
- **语言识别**: `SpokenLanguageIdentification`
- **音频标注**: `AudioTagging`
- **标点恢复**: `OfflinePunctuation`, `OnlinePunctuation`

## 关键依赖与配置

### 依赖项
- **Java**: 1.8+
- **ONNX Runtime**: 需要系统安装
- **Sherpa-ONNX JNI**: `libsherpa-onnx-jni`

### 配置系统
每个功能都有对应的 Config 类，支持：
- 模型路径配置
- 解码参数配置
- 特征提取配置
- 设备配置（CPU/GPU）

### 库加载
```java
// 自动加载本地库
LibraryLoader.loadLibrary();

// 或手动指定路径
System.load("/path/to/libsherpa-onnx-jni.dylib");
```

## 数据模型

### 主要数据类
- `OfflineRecognizerResult` - 识别结果
- `OnlineRecognizerResult` - 在线识别结果
- `GeneratedAudio` - 合成的音频
- `SpeechSegment` - 语音片段
- `AudioEvent` - 音频事件
- `SpeakerEmbedding` - 说话人嵌入

### 音频处理
- `WaveReader` - WAV 文件读取
- `WaveWriter` - WAV 文件写入
- 支持多种采样率和位深度

## 测试与质量

### 测试覆盖
- 单元测试：未提供
- 集成测试：通过示例项目验证
- 性能测试：未包含

### 常见问题
1. **JNI 库加载失败**
   - 确保 `libsherpa-onnx-jni` 在系统路径中
   - 检查 ONNX Runtime 是否正确安装

2. **模型加载错误**
   - 验证模型文件路径
   - 检查模型文件完整性

3. **内存泄漏**
   - 及时释放 `OnlineStream` 等资源
   - 使用 try-with-resources 模式

## 部署说明

### 平台支持
- Windows x64
- macOS x64/ARM64
- Linux x64

### 打包方式
1. **Maven 依赖**: 从中央仓库获取
2. **本地构建**: 使用 `mvn package` 或 `make`
3. **动态库**: 需要与平台匹配的 JNI 库

### 系统要求
- Java 8+
- ONNX Runtime 1.17+
- 足够的内存用于模型加载

## 相关文件清单

### 核心实现
- `src/main/java/com/k2fsa/sherpa/onnx/` - 所有 Java 类
- `pom.xml` - Maven 配置
- `Makefile` - 构建脚本

### 文档
- `readme.md` - 英文使用指南
- `readme.zh.md` - 中文使用指南
- `.build.txt` - 构建说明

### 资源
- `src/main/resources/` - 资源文件

## 变更记录 (Changelog)

### 2025-12-18 10:45:53
- ✨ 创建 Java API 模块文档
- 📊 初始文档覆盖率 0%
- 📝 添加 API 接口说明和使用示例

---

*更多详细信息请参考：https://github.com/k2-fsa/sherpa-onnx/tree/master/sherpa-onnx/java-api*