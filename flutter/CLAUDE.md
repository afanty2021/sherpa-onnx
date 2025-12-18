[根目录](../../CLAUDE.md) > **flutter**

# Sherpa-ONNX Flutter SDK

> 更新时间：2025-12-18 10:45:53

## 模块职责

Flutter SDK 模块为 Sherpa-ONNX 提供了完整的 Flutter 插件支持，使得 Flutter 应用能够跨平台使用所有语音处理功能。该模块采用联邦插件（Federated Plugin）架构，支持 Android、iOS、Windows、macOS 和 Linux 平台。

## 入口与启动

### 主要组件
- **核心包**: `sherpa_onnx` (Dart API 定义)
- **平台实现**:
  - `sherpa_onnx_android` - Android 实现
  - `sherpa_onnx_ios` - iOS 实现
  - `sherpa_onnx_macos` - macOS 实现
  - `sherpa_onnx_windows` - Windows 实现
  - `sherpa_onnx_linux` - Linux 实现

### 版本信息
- **当前版本**: 1.12.20
- **Dart SDK**: >=3.1.0 <4.0.0
- **Flutter**: >=2.8.1

## 对外接口

### 1. 语音识别 (ASR)
```dart
// 离线识别
final modelConfig = OfflineModelConfig(
  paraformer: OfflineParaformerModelConfig(
    model: 'assets/paraformer/model.onnx',
  ),
  tokens: 'assets/tokens.txt',
  numThreads: 2,
);

final config = OfflineRecognizerConfig(
  modelConfig: modelConfig,
  decodingMethod: 'greedy_search',
);

final recognizer = await OfflineRecognizer.create(config);
final result = await recognizer.decodeFile('test.wav');
print(result.text);
```

### 2. 在线识别
```dart
// 在线识别
final onlineModelConfig = OnlineModelConfig(
  transducer: OnlineTransducerModelConfig(
    encoder: 'assets/encoder.onnx',
    decoder: 'assets/decoder.onnx',
    joiner: 'assets/joiner.onnx',
  ),
);

final onlineConfig = OnlineRecognizerConfig(
  modelConfig: onlineModelConfig,
);

final onlineRecognizer = await OnlineRecognizer.create(onlineConfig);
final stream = onlineRecognizer.createStream();

// 实时处理音频
stream.acceptWaveform(audioSamples);
final result = onlineRecognizer.decode(stream);
```

### 3. 语音合成 (TTS)
```dart
final ttsModelConfig = OfflineTtsModelConfig(
  vits: OfflineTtsVitsModelConfig(
    model: 'assets/vits/model.onnx',
  ),
  numThreads: 1,
);

final ttsConfig = OfflineTtsConfig(
  modelConfig: ttsModelConfig,
  ruleFsts: '',
  maxNumSentences: 2,
);

final tts = await OfflineTts.create(ttsConfig);
final audio = await tts.generate('Hello, world!');
await WaveWriter.write('output.wav', audio.samples, audio.sampleRate);
```

### 4. 说话人识别
```dart
final extractorConfig = SpeakerEmbeddingExtractorConfig(
  modelConfig: OfflineSpeakerEmbeddingExtractorModelConfig(
    model: 'assets/cam++.onnx',
  ),
);

final extractor = await SpeakerEmbeddingExtractor.create(extractorConfig);
final embedding = await extractor.extract('test.wav');

final manager = SpeakerEmbeddingManager();
manager.register('speaker1', embedding);
final name = manager.search(embedding);
```

## 关键依赖与配置

### 核心依赖
- **ffi**: Dart FFI 用于调用原生代码
- **Flutter SDK**: UI 框架
- **平台特定插件**: 各平台的原生实现

### 模型配置
支持多种模型类型：
- **Paraformer**: 阿里巴巴 ASR 模型
- **Transducer**: RNN-T 模型
- **Whisper**: OpenAI 多语言模型
- **VITS**: 语音合成模型
- **CAM++**: 说话人识别模型

### 平台特定要求
- **Android**: API 21+
- **iOS**: iOS 11.0+
- **Windows**: Windows 10+
- **macOS**: macOS 10.14+
- **Linux**: GLIBC 2.17+

## 数据模型

### 主要数据类
- `OfflineRecognizerResult` - 离线识别结果
- `OnlineRecognizerResult` - 在线识别结果
- `GeneratedAudio` - 合成的音频
- `SpeakerEmbedding` - 说话人嵌入
- `SpeechSegment` - 语音片段

### 音频处理
- `WaveReader` - WAV 文件读取
- `WaveWriter` - WAV 文件写入
- 支持多种采样率和格式

## 构建与发布

### 开发环境设置
```bash
# 克隆项目
git clone https://github.com/k2-fsa/sherpa-onnx.git
cd sherpa-onnx/flutter

# 获取依赖
flutter pub get

# 运行示例
cd sherpa_onnx/example
flutter run
```

### 发布流程
1. 更新所有子包的版本号
2. 运行测试确保功能正常
3. 发布到 pub.dev：
```bash
flutter pub publish --dry-run
flutter pub publish
```

### 平台构建
- **Android**: 自动配置，支持 arm64-v8a、armeabi-v7a、x86_64
- **iOS**: 通过 CocoaPods 管理
- **Windows**: 使用 CMake 构建
- **Linux**: 预编译二进制库
- **macOS**: 通过 CocoaPods 管理

## 测试与质量

### 测试覆盖
- 单元测试：每个平台都有基础测试
- 集成测试：通过示例应用验证
- 端到端测试：跨平台功能验证

### 性能优化
- 使用 FFI 减少桥接开销
- 异步处理避免阻塞 UI
- 内存池管理减少 GC 压力
- 模型量化减少内存占用

### 常见问题
1. **模型文件加载失败**
   - 确保模型文件在 assets 目录
   - 检查 pubspec.yaml 配置

2. **权限问题**
   - Android: 在 AndroidManifest.xml 添加录音权限
   - iOS: 在 Info.plist 添加麦克风使用说明

3. **平台兼容性**
   - 确保目标平台支持所需的模型格式
   - 检查最低系统版本要求

## 示例应用

### 基础语音识别
```dart
import 'package:sherpa_onnx/sherpa_onnx.dart';

class ASRApp extends StatefulWidget {
  @override
  _ASRAppState createState() => _ASRAppState();
}

class _ASRAppState extends State<ASRApp> {
  late OfflineRecognizer _recognizer;
  String _result = '';

  @override
  void initState() {
    super.initState();
    _initRecognizer();
  }

  Future<void> _initRecognizer() async {
    final config = getOfflineRecognizerConfig();
    _recognizer = await OfflineRecognizer.create(config);
  }

  Future<void> _recognizeFile() async {
    final result = await _recognizer.decodeFile('test.wav');
    setState(() {
      _result = result.text;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Sherpa-ONNX Flutter')),
      body: Center(
        child: Column(
          children: [
            ElevatedButton(
              onPressed: _recognizeFile,
              child: Text('识别音频文件'),
            ),
            Text('识别结果: $_result'),
          ],
        ),
      ),
    );
  }
}
```

## 相关文件清单

### 核心文件
- `sherpa_onnx/lib/` - Dart API 实现
- `sherpa_onnx/pubspec.yaml` - 主包配置
- `sherpa_onnx_android/` - Android 平台实现
- `sherpa_onnx_ios/` - iOS 平台实现
- `sherpa_onnx_macos/` - macOS 平台实现
- `sherpa_onnx_windows/` - Windows 平台实现
- `sherpa_onnx_linux/` - Linux 平台实现

### 文档
- `README.md` - 项目说明
- `notes.md` - 开发笔记
- `publish.md` - 发布指南

### 示例
- `sherpa_onnx/example/` - 示例应用

## 变更记录 (Changelog)

### 2025-12-18 10:45:53
- ✨ 创建 Flutter SDK 模块文档
- 📊 初始文档覆盖率 0%
- 📝 添加 API 接口说明和示例代码

---

*更多信息和问题反馈请访问：https://pub.dev/packages/sherpa_onnx*