[根目录](../CLAUDE.md) > **flutter-examples**

# Flutter 示例

> 更新时间：2025-12-10 07:48:28

## 模块职责

Flutter 示例模块展示了如何在跨平台移动应用中集成 Sherpa-ONNX 的语音处理功能。这些示例使用 Dart 语言和 Flutter 框架，提供了在 iOS、Android、Windows、macOS 和 Linux 上一致的语音 AI 体验。

## 示例应用概览

### 1. 实时语音识别 (Streaming ASR)
**路径**: `streaming_asr/`

**功能特点**:
- 实时麦克风输入处理
- 流式语音识别
- 可视化识别结果
- 支持多种模型切换

**核心文件**:
- `lib/main.dart` - 应用入口
- `lib/streaming_asr.dart` - 识别界面
- `lib/online_model.dart` - 模型配置
- `lib/utils.dart` - 工具函数

### 2. 语音合成 (TTS)
**路径**: `tts/`

**功能特点**:
- 文本转语音
- 多种 TTS 模型支持
- 音频播放功能
- Isolate 隔离处理

**核心文件**:
- `lib/tts.dart` - TTS 主界面
- `lib/isolate_tts.dart` - 后台 TTS 处理
- `lib/offline_model.dart` - TTS 模型配置

### 3. 非流式 VAD+ASR
**路径**: `non_streaming_vad_asr/`

**功能特点**:
- VAD（语音活动检测）
- 非流式文件识别
- 批处理音频文件

## 入口与启动

### 环境要求
- **Flutter SDK**: 3.0+
- **Dart SDK**: 2.17+
- **平台**: iOS 13+, Android API 21+

### 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/k2-fsa/sherpa-onnx.git
cd sherpa-onnx/flutter-examples/streaming_asr

# 2. 安装依赖
flutter pub get

# 3. 运行应用
flutter run

# 4. 指定平台运行
flutter run -d ios
flutter run -d android
flutter run -d windows
flutter run -d macos
```

### 依赖配置

在 `pubspec.yaml` 中添加：
```yaml
dependencies:
  flutter:
    sdk: flutter

  # Sherpa-ONNX Flutter 插件
  sherpa_onnx: ^1.0.0

  # 音频处理
  record: ^5.0.4
  audioplayers: ^5.2.1

  # 路径和文件
  path_provider: ^2.1.1
  path: ^1.8.3

  # 状态管理
  provider: ^6.1.1
```

## 对外接口

### 1. 实时语音识别接口

#### 初始化识别器
```dart
import 'package:sherpa_onnx/sherpa_onnx.dart' as sherpa_onnx;

// 创建在线识别器
Future<sherpa_onnx.OnlineRecognizer> createOnlineRecognizer() async {
  // 配置模型
  final modelConfig = sherpa_onnx.OnlineModelConfig(
    zipformer: sherpa_onnx.OnlineZipformerModelConfig(
      encoder: 'models/encoder.onnx',
      decoder: 'models/decoder.onnx',
      joiner: 'models/joiner.onnx',
    ),
    tokens: 'models/tokens.txt',
    numThreads: 2,
    provider: 'cpu',
    debug: false,
  );

  // 配置识别器
  final config = sherpa_onnx.OnlineRecognizerConfig(
    model: modelConfig,
    featConfig: sherpa_onnx.FeatureConfig(
      sampleRate: 16000,
      featureDim: 80,
    ),
    endpointConfig: sherpa_onnx.EndpointConfig(
      rule1MinTrailingSilence: 2.4,
      rule2MinTrailingSilence: 0.8,
      rule3MinUtteranceLength: 30,
    ),
  );

  return sherpa_onnx.OnlineRecognizer(config);
}
```

#### 实时音频处理
```dart
class _StreamingAsrScreenState extends State<StreamingAsrScreen> {
  late final AudioRecorder _audioRecorder;
  sherpa_onnx.OnlineRecognizer? _recognizer;
  sherpa_onnx.OnlineStream? _stream;

  @override
  void initState() {
    super.initState();
    _audioRecorder = AudioRecorder();

    // 初始化识别器
    _initRecognizer();
  }

  Future<void> _startRecording() async {
    // 配置录音
    const config = RecordConfig(
      encoder: AudioEncoder.pcm16bits,
      sampleRate: 16000,
      numChannels: 1,
    );

    // 开始录音流
    final stream = await _audioRecorder.startStream(config);

    // 处理音频数据
    stream.listen((data) {
      final samples = convertBytesToFloat32(data);

      // 提交给识别器
      _stream?.acceptWaveform(
        samples: samples,
        sampleRate: 16000,
      );

      // 获取识别结果
      while (_recognizer?.isReady(_stream) == true) {
        _recognizer?.decode(_stream);
      }

      final result = _recognizer?.getResult(_stream);
      if (result?.text.isNotEmpty == true) {
        setState(() {
          _textController.text = result!.text;
        });
      }
    });
  }
}
```

### 2. 语音合成接口

#### TTS 初始化
```dart
import 'package:sherpa_onnx/sherpa_onnx.dart' as sherpa_onnx;

Future<sherpa_onnx.OfflineTts> createTts() async {
  // 配置 VITS 模型
  final modelConfig = sherpa_onnx.OfflineTtsModelConfig(
    vits: sherpa_onnx.OfflineTtsVitsModelConfig(
      model: 'models/vits.onnx',
      lexicon: 'models/lexicon.txt',
    ),
    tokens: 'models/tokens.txt',
    numThreads: 1,
    debug: true,
  );

  // 配置 TTS
  final config = sherpa_onnx.OfflineTtsConfig(
    model: modelConfig,
    ruleFsts: '',
    maxNumSentences: 2,
  );

  return sherpa_onnx.OfflineTts(config);
}
```

#### 文本转语音
```dart
class TtsScreen extends StatefulWidget {
  @override
  _TtsScreenState createState() => _TtsScreenState();
}

class _TtsScreenState extends State<TtsScreen> {
  sherpa_onnx.OfflineTts? _tts;
  final AudioPlayer _player = AudioPlayer();

  Future<void> _generateSpeech(String text) async {
    // 生成语音
    final audio = _tts?.generate(
      text: text,
      speakerId: 0,
      speed: 1.0,
    );

    if (audio != null) {
      // 保存临时文件
      final tempDir = await getTemporaryDirectory();
      final file = File('${tempDir.path}/output.wav');
      await file.writeAsBytes(audio.samples);

      // 播放音频
      await _player.play(DeviceFileSource(file.path));
    }
  }
}
```

### 3. Isolate 隔离处理

#### 使用 Isolate 避免阻塞 UI
```dart
import 'dart:isolate';

// TTS Isolate 入口点
void _ttsIsolateEntry(SendPort sendPort) {
  final port = ReceivePort();
  sendPort.send(port.sendPort);

  port.listen((message) async {
    if (message is Map<String, dynamic>) {
      final text = message['text'] as String;
      final replyPort = message['replyPort'] as SendPort;

      // 在 Isolate 中生成语音
      final audio = await generateTtsInIsolate(text);

      replyPort.send(audio);
    }
  });
}

// 主类中使用 Isolate
class IsolateTtsView extends StatefulWidget {
  @override
  _IsolateTtsViewState createState() => _IsolateTtsViewState();
}

class _IsolateTtsViewState extends State<IsolateTtsView> {
  Isolate? _ttsIsolate;
  SendPort? _ttsSendPort;

  Future<void> _initIsolate() async {
    final receivePort = ReceivePort();
    _ttsIsolate = await Isolate.spawn(_ttsIsolateEntry, receivePort.sendPort);

    _ttsSendPort = await receivePort.first as SendPort;
  }

  Future<void> _generateSpeechIsolate(String text) async {
    final responsePort = ReceivePort();

    _ttsSendPort?.send({
      'text': text,
      'replyPort': responsePort.sendPort,
    });

    // 等待结果
    final audio = await responsePort.first;

    // 播放音频
    _playAudio(audio);
  }
}
```

## 关键依赖与配置

### Flutter 插件依赖

```yaml
# pubspec.yaml
dependencies:
  sherpa_onnx: ^1.0.0          # Sherpa-ONNX Flutter 插件

  # 音频录制
  record: ^5.0.4               # 跨平台录音

  # 音频播放
  audioplayers: ^5.2.1         # 音频播放器
  just_audio: ^0.9.36          # 另一个音频播放选项

  # 文件系统
  path_provider: ^2.1.1        # 路径管理
  path: ^1.8.3                 # 路径操作

  # 工具库
  permission_handler: ^11.0.1  # 权限管理
  file_picker: ^6.1.1          # 文件选择
```

### 平台特定配置

#### Android (`android/app/src/main/AndroidManifest.xml`)
```xml
<!-- 麦克风权限 -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

<!-- 网络权限（下载模型） -->
<uses-permission android:name="android.permission.INTERNET" />
```

#### iOS (`ios/Runner/Info.plist`)
```xml
<key>NSMicrophoneUsageDescription</key>
<string>此应用需要访问麦克风进行语音识别</string>

<key>NSDocumentsFolderUsageDescription</key>
<string>此应用需要访问文档文件夹加载音频文件</string>
```

#### Windows (`windows/runner/CMakeLists.txt`)
```cmake
# 添加 Sherpa-ONNX 依赖
find_package(sherpa-onnx REQUIRED CONFIG)

target_link_libraries(${BINARY_NAME} PRIVATE
  sherpa-onnx::sherpa-onnx
  flutter
  flutter_wrapper_app
)
```

### 模型文件管理

```dart
class ModelManager {
  static Future<String> _downloadModel(String url, String filename) async {
    final directory = await getApplicationDocumentsDirectory();
    final file = File('${directory.path}/$filename');

    if (await file.exists()) {
      return file.path;
    }

    final request = HttpRequest.request(
      url,
      method: 'GET',
    );

    final response = await request;
    await file.writeAsBytes(response.responseBytes);

    return file.path;
  }

  static Future<void> initializeModels() async {
    // 下载 ASR 模型
    final asrModelUrl = 'https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-zipformer-zh-14M-2023-02-23.tar.bz2';
    await _downloadModel(asrModelUrl, 'asr_model.onnx');

    // 下载 TTS 模型
    final ttsModelUrl = 'https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-piper-zh_CN-huayan-medium.tar.bz2';
    await _downloadModel(ttsModelUrl, 'tts_model.onnx');
  }
}
```

## 性能优化

### 1. 音频缓冲区优化

```dart
class AudioProcessor {
  static const int _bufferSize = 4096;
  static const int _sampleRate = 16000;

  late final List<double> _audioBuffer;
  int _bufferIndex = 0;

  void processAudioChunk(List<int> chunk) {
    final samples = convertBytesToFloat32(chunk);

    for (final sample in samples) {
      _audioBuffer[_bufferIndex] = sample;
      _bufferIndex++;

      if (_bufferIndex >= _bufferSize) {
        // 处理满缓冲区
        _processBuffer();
        _bufferIndex = 0;
      }
    }
  }
}
```

### 2. 内存管理

```dart
class RecognizerManager {
  sherpa_onnx.OnlineRecognizer? _recognizer;

  // 使用资源池管理识别器
  final List<sherpa_onnx.OnlineRecognizer> _pool = [];
  final int _maxPoolSize = 3;

  sherpa_onnx.OnlineRecognizer getRecognizer() {
    if (_pool.isNotEmpty) {
      return _pool.removeLast();
    }

    return _createNewRecognizer();
  }

  void returnRecognizer(sherpa_onnx.OnlineRecognizer recognizer) {
    if (_pool.length < _maxPoolSize) {
      _pool.add(recognizer);
    } else {
      recognizer.dispose();
    }
  }
}
```

### 3. UI 响应性优化

```dart
// 使用 Compute 分离计算密集型任务
Future<List<double>> computeAudioFeatures(List<double> audioData) async {
  return await compute(_extractFeatures, audioData);
}

// Isolate 中的计算函数
List<double> _extractFeatures(List<double> audioData) {
  // 执行特征提取
  // ...
  return features;
}
```

## 测试策略

### 1. 单元测试

```dart
// test/asr_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:sherpa_onnx/sherpa_onnx.dart';

void main() {
  group('ASR Tests', () {
    test('Model loading', () async {
      final recognizer = await createTestRecognizer();
      expect(recognizer, isNotNull);
    });

    test('Audio processing', () {
      final testData = generateTestAudio();
      final result = processAudio(testData);
      expect(result, isNotEmpty);
    });
  });
}
```

### 2. Widget 测试

```dart
// test/widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('ASR UI test', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());

    // 查找开始按钮
    final startButton = find.widgetWithText(ElevatedButton, 'Start');
    expect(startButton, findsOneWidget);

    // 点击按钮
    await tester.tap(startButton);
    await tester.pump();

    // 验证状态变化
    expect(find.text('Recording...'), findsOneWidget);
  });
}
```

### 3. 集成测试

```dart
// integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('End-to-end ASR flow', (WidgetTester tester) async {
    app.main();
    await tester.pumpAndSettle();

    // 执行完整识别流程
    await tester.tap(find.byKey(Key('start_button')));
    await tester.pumpAndSettle(Duration(seconds: 5));

    // 验证结果
    expect(find.byType(Text), findsWidgets);
  });
}
```

## 部署指南

### 1. Android 打包

```bash
# 生成签名密钥
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload

# 配置签名 (android/key.properties)
storePassword=myStorePassword
keyPassword=myKeyPassword
keyAlias=myKeyAlias
storeFile=myStoreFileLocation

# 构建 Release APK
flutter build apk --release

# 构建 AAB
flutter build appbundle --release
```

### 2. iOS 打包

```bash
# 配置 iOS 证书和描述文件
open ios/Runner.xcworkspace

# 构建
flutter build ios --release

# 使用 Xcode Archive
# Product > Archive
```

### 3. 模型文件优化

```dart
class AssetBundleHelper {
  static Future<void> preloadModels() async {
    // 预加载模型到内存
    final manifest = await DefaultAssetBundle.of(context).loadString('AssetManifest.json');
    final assets = json.decode(manifest) as Map<String, dynamic>;

    final modelAssets = assets.keys
        .where((path) => path.contains('models'))
        .toList();

    for (final asset in modelAssets) {
      await DefaultAssetBundle.of(context).load(asset);
    }
  }
}
```

## 常见问题 (FAQ)

### Q1: Flutter 应用启动慢？
A: 优化方案：
- 延迟加载模型
- 使用 Isolate 预加载
- 压缩模型文件

### Q2: 实时识别卡顿？
A: 解决方案：
- 调整缓冲区大小
- 使用 compute 分离计算
- 优化音频处理流程

### Q3: Android 权限问题？
A: 处理方式：
```dart
import 'package:permission_handler/permission_handler.dart';

Future<void> requestPermissions() async {
  final status = await Permission.microphone.request();
  if (status.isDenied) {
    // 显示权限说明
    openAppSettings();
  }
}
```

### Q4: 模型文件太大？
A: 解决方案：
- 使用动态下载
- 实现模型压缩
- 按需加载不同语言模型

### Q5: iOS 后台录音问题？
A: 配置 Info.plist：
```xml
<key>UIBackgroundModes</key>
<array>
    <string>audio</string>
</array>
```

## 相关文件清单

### 示例应用
- `streaming_asr/` - 实时语音识别
- `tts/` - 语音合成
- `non_streaming_vad_asr/` - VAD+ASR 组合

### 核心文件
- `lib/main.dart` - 应用入口
- `lib/asr.dart` - ASR 实现
- `lib/tts.dart` - TTS 实现
- `lib/model.dart` - 模型配置
- `pubspec.yaml` - 依赖配置

### 平台配置
- `android/` - Android 配置
- `ios/` - iOS 配置
- `windows/` - Windows 配置
- `macos/` - macOS 配置
- `linux/` - Linux 配置

## 变更记录

### 2025-12-10 07:48:28
- 📝 创建 Flutter 示例文档
- 📱 添加跨平台支持说明
- 🔧 补充性能优化技巧
- 💡 提供部署和测试指南

---

*相关文件：[Flutter 插件](../flutter/sherpa_onnx_ios/CLAUDE.md) | [Android 集成](../android/CLAUDE.md) | [iOS 集成](../ios/CLAUDE.md)*