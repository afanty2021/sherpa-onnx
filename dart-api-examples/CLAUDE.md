# Dart API 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / Dart API 示例

## 模块概述

本模块包含 Sherpa-ONNX Dart 语言绑定的使用示例。Dart 是由 Google 开发的现代编程语言，特别适合构建移动应用（Flutter）、Web 应用和服务器端应用。

## 环境要求

- Dart SDK 2.17 或更高版本
- Flutter SDK（用于移动应用）
- ONNX Runtime 库

## 快速开始

### 1. 安装 Dart
```bash
# macOS
brew tap dart-lang/dart
brew install dart

# Linux
sudo apt-get update
sudo apt-get install dart

# Windows
# 从 https://dart.dev/get-dart 下载安装
```

### 2. 创建新项目
```bash
dart create sherpa_onnx_example
cd sherpa_onnx_example
```

### 3. 添加依赖
在 `pubspec.yaml` 中添加：
```yaml
dependencies:
  sherpa_onnx: ^0.1.0
  path: ^1.8.0
```

## 使用示例

### 基本语音识别

```dart
// bin/asr.dart
import 'dart:io';
import 'package:sherpa_onnx/sherpa_onnx.dart';

void main() async {
  // 配置模型
  final modelConfig = OfflineModelConfig(
    transducer: OfflineTransducerModelConfig(
      encoder: 'models/encoder.onnx',
      decoder: 'models/decoder.onnx',
      joiner: 'models/joiner.onnx',
    ),
    tokens: 'models/tokens.txt',
    numThreads: 2,
    modelType: 'zipformer',
  );

  // 创建识别器
  final recognizer = OfflineRecognizer(modelConfig);

  // 识别音频文件
  final audioFile = 'test.wav';
  final waveData = await readWaveFile(audioFile);

  final stream = recognizer.createStream();
  stream.acceptWaveform(waveData.samples, waveData.sampleRate);

  // 执行识别
  recognizer.decode(stream);
  final result = recognizer.getResult(stream);

  print('识别结果: ${result.text}');

  // 清理资源
  recognizer.dispose();
  stream.dispose();
}
```

### 实时语音识别

```dart
import 'dart:async';
import 'dart:typed_data';
import 'package:sherpa_onnx/sherpa_onnx.dart';

class RealTimeASR {
  late OnlineRecognizer recognizer;
  late StreamController<RecognitionResult> resultController;
  bool isRunning = false;

  void initialize() {
    final modelConfig = OnlineModelConfig(
      transducer: OnlineTransducerModelConfig(
        encoder: 'models/encoder.onnx',
        decoder: 'models/decoder.onnx',
        joiner: 'models/joiner.onnx',
      ),
      tokens: 'models/tokens.txt',
      numThreads: 1,
    );

    final recognizerConfig = OnlineRecognizerConfig(
      modelConfig: modelConfig,
      enableEndpoint: true,
      rule1MinSilenceDuration: 0.5,
    );

    recognizer = OnlineRecognizer(recognizerConfig);
    resultController = StreamController.broadcast();
  }

  void startRecognition() {
    if (!isRunning) {
      isRunning = true;
      _processAudioStream();
    }
  }

  void stopRecognition() {
    isRunning = false;
  }

  Stream<RecognitionResult> get results => resultController.stream;

  Future<void> _processAudioStream() async {
    final stream = recognizer.createStream();

    while (isRunning) {
      // 模拟音频输入
      final audioData = await captureAudio();

      if (audioData.isNotEmpty) {
        stream.acceptWaveform(audioData, 16000);
        recognizer.decode(stream);

        final result = recognizer.getResult(stream);
        if (result.text.isNotEmpty) {
          resultController.add(result);
        }
      }

      await Future.delayed(Duration(milliseconds: 100));
    }

    stream.dispose();
  }

  Future<Float32List> captureAudio() async {
    // 实现音频捕获逻辑
    return Float32List(1600); // 返回100ms的音频数据
  }
}

void main() async {
  final asr = RealTimeASR();
  asr.initialize();

  // 监听识别结果
  asr.results.listen((result) {
    print('实时识别: ${result.text}');
  });

  asr.startRecognition();

  // 运行30秒后停止
  await Future.delayed(Duration(seconds: 30));
  asr.stopRecognition();
}
```

### 语音合成

```dart
import 'dart:io';
import 'package:sherpa_onnx/sherpa_onnx.dart';

void main() async {
  // 配置 TTS 模型
  final modelConfig = OfflineTtsModelConfig(
    vits: OfflineTtsVitsModelConfig(
      model: 'models/vits.onnx',
      lexicon: 'models/lexicon.txt',
      tokens: 'models/tokens.txt',
    ),
    numThreads: 1,
  );

  // 创建 TTS 模型
  final model = OfflineTtsModel(modelConfig);

  // 生成语音
  final text = 'Hello, world! 你好，世界！';
  final audio = model.generate(text, sid: 0);

  // 保存音频文件
  final outputFile = 'output.wav';
  final file = File(outputFile);
  await file.writeAsBytes(audio.wavData);

  print('语音已保存到: $outputFile');
  print('采样率: ${audio.sampleRate} Hz');
  print('时长: ${audio.duration} 秒');

  model.dispose();
}
```

### 说话人识别

```dart
import 'dart:math';
import 'package:sherpa_onnx/sherpa_onnx.dart';

double cosineSimilarity(List<double> x, List<double> y) {
  double dot = 0.0;
  double normX = 0.0;
  double normY = 0.0;

  for (int i = 0; i < x.length; i++) {
    dot += x[i] * y[i];
    normX += x[i] * x[i];
    normY += y[i] * y[i];
  }

  return dot / (sqrt(normX) * sqrt(normY));
}

void main() async {
  // 配置说话人模型
  final modelConfig = SpeakerEmbeddingExtractorModelConfig(
    model: 'models/speaker_model.onnx',
    numThreads: 1,
  );

  // 创建提取器
  final extractor = SpeakerEmbeddingExtractor(modelConfig);

  // 提取两个音频的说话人特征
  final audio1 = 'speaker1.wav';
  final audio2 = 'speaker2.wav';

  final embedding1 = await extractor.compute(audio1);
  final embedding2 = await extractor.compute(audio2);

  // 计算相似度
  final similarity = cosineSimilarity(embedding1, embedding2);

  print('说话人相似度: ${similarity.toStringAsFixed(2)}');
  if (similarity > 0.5) {
    print('判断为同一说话人');
  } else {
    print('判断为不同说话人');
  }

  extractor.dispose();
}
```

## Flutter 集成

### 1. 创建 Flutter 项目
```bash
flutter create sherpa_onnx_flutter
cd sherpa_onnx_flutter
```

### 2. 添加插件依赖
在 `pubspec.yaml` 中：
```yaml
dependencies:
  flutter:
    sdk: flutter

  sherpa_onnx: ^0.1.0
  permission_handler: ^10.0.0
  record: ^4.4.4
  path_provider: ^2.0.0
```

### 3. 实现 Flutter 应用

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:sherpa_onnx/sherpa_onnx.dart';
import 'package:record/record.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Sherpa-ONNX Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const SpeechRecognitionPage(),
    );
  }
}

class SpeechRecognitionPage extends StatefulWidget {
  const SpeechRecognitionPage({Key? key}) : super(key: key);

  @override
  State<SpeechRecognitionPage> createState() => _SpeechRecognitionPageState();
}

class _SpeechRecognitionPageState extends State<SpeechRecognitionPage> {
  late OnlineRecognizer recognizer;
  final AudioRecorder recorder = AudioRecorder();
  bool isRecording = false;
  String resultText = '';

  @override
  void initState() {
    super.initState();
    _initializeRecognizer();
  }

  void _initializeRecognizer() {
    final modelConfig = OnlineModelConfig(
      transducer: OnlineTransducerModelConfig(
        encoder: 'models/encoder.onnx',
        decoder: 'models/decoder.onnx',
        joiner: 'models/joiner.onnx',
      ),
      tokens: 'models/tokens.txt',
      numThreads: 1,
    );

    final recognizerConfig = OnlineRecognizerConfig(
      modelConfig: modelConfig,
      enableEndpoint: true,
    );

    recognizer = OnlineRecognizer(recognizerConfig);
  }

  Future<void> _startRecording() async {
    try {
      setState(() {
        isRecording = true;
        resultText = '';
      });

      // 开始录音
      await recorder.start(const RecordConfig(
        encoder: AudioEncoder.pcm16bit,
        sampleRate: 16000,
        bitRate: 128000,
      ));

      // 处理音频流
      final stream = await recorder.startStream(const RecordConfig(
        encoder: AudioEncoder.pcm16bit,
        sampleRate: 16000,
      ));

      stream.listen((data) {
        _processAudio(data);
      });
    } catch (e) {
      setState(() {
        isRecording = false;
      });
      print('录音失败: $e');
    }
  }

  Future<void> _stopRecording() async {
    try {
      await recorder.stop();
      setState(() {
        isRecording = false;
      });
    } catch (e) {
      print('停止录音失败: $e');
    }
  }

  void _processAudio(Uint8List data) {
    // 将字节数据转换为 float32
    final samples = Float32List(data.length ~/ 2);
    for (int i = 0; i < samples.length; i++) {
      final byte1 = data[i * 2];
      final byte2 = data[i * 2 + 1];
      final int16 = (byte2 << 8) | byte1;
      samples[i] = int16 / 32768.0;
    }

    // 识别
    final stream = recognizer.createStream();
    stream.acceptWaveform(samples, 16000);
    recognizer.decode(stream);

    final result = recognizer.getResult(stream);
    if (result.text.isNotEmpty) {
      setState(() {
        resultText = result.text;
      });
    }

    stream.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Sherpa-ONNX Speech Recognition'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            ElevatedButton(
              onPressed: isRecording ? _stopRecording : _startRecording,
              child: Text(isRecording ? '停止录音' : '开始录音'),
            ),
            const SizedBox(height: 20),
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text(
                resultText.isEmpty ? '识别结果将显示在这里...' : resultText,
                style: const TextStyle(fontSize: 18),
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    recognizer.dispose();
    super.dispose();
  }
}
```

## 服务器端使用

### 使用 Aqueduct 框架

```dart
// lib/server.dart
import 'package:aqueduct/aqueduct.dart';
import 'package:sherpa_onnx/sherpa_onnx.dart';

class ASRController extends ResourceController {
  late OfflineRecognizer recognizer;

  @override
  Future<void> didOpen() async {
    final modelConfig = OfflineModelConfig(
      transducer: OfflineTransducerModelConfig(
        encoder: 'models/encoder.onnx',
        decoder: 'models/decoder.onnx',
        joiner: 'models/joiner.onnx',
      ),
      tokens: 'models/tokens.txt',
    );

    recognizer = OfflineRecognizer(modelConfig);
  }

  @override
  Future<void> didClose() async {
    recognizer.dispose();
  }

  @Operation.post()
  Future<Response> recognize() async {
    final body = await request.body.decode<List<int>>();

    // 转换音频数据
    final samples = Float32List(body.length ~/ 2);
    for (int i = 0; i < samples.length; i++) {
      final byte1 = body[i * 2];
      final byte2 = body[i * 2 + 1];
      final int16 = (byte2 << 8) | byte1;
      samples[i] = int16 / 32768.0;
    }

    // 识别
    final stream = recognizer.createStream();
    stream.acceptWaveform(samples, 16000);
    recognizer.decode(stream);
    final result = recognizer.getResult(stream);

    return Response.ok({'text': result.text});
  }
}

class SherpaOnnxChannel extends ApplicationChannel {
  @override
  Controller get entryPoint {
    final router = Router();
    router.route('/recognize').link(() => ASRController());
    return router;
  }
}
```

## 性能优化

### 1. 使用 Isolate 进行后台处理

```dart
import 'dart:isolate';

void recognitionIsolate(SendPort sendPort) {
  final port = ReceivePort();
  sendPort.send(port.sendPort);

  final recognizer = OfflineRecognizer(modelConfig);

  port.listen((message) {
    if (message['action'] == 'recognize') {
      final samples = message['samples'] as Float32List;
      final stream = recognizer.createStream();
      stream.acceptWaveform(samples, 16000);
      recognizer.decode(stream);
      final result = recognizer.getResult(stream);
      sendPort.send({'text': result.text});
      stream.dispose();
    }
  });
}

Future<String> recognizeInIsolate(Float32List samples) async {
  final receivePort = ReceivePort();
  await Isolate.spawn(recognitionIsolate, receivePort.sendPort);

  final sendPort = await receivePort.first as SendPort;
  final responsePort = ReceivePort();
  sendPort.send({'port': responsePort.sendPort});

  final isolateSendPort = await responsePort.first as SendPort;
  final answerPort = ReceivePort();
  isolateSendPort.send({
    'action': 'recognize',
    'samples': samples,
    'port': answerPort.sendPort,
  });

  final result = await answerPort.first as Map;
  return result['text'] as String;
}
```

### 2. 音频缓冲管理

```dart
class AudioBuffer {
  final int bufferSize;
  final Float32List buffer;
  int _position = 0;

  AudioBuffer(this.bufferSize) : buffer = Float32List(bufferSize);

  void addSamples(Float32List samples) {
    for (int i = 0; i < samples.length && _position < buffer.length; i++) {
      buffer[_position++] = samples[i];
    }
  }

  Float32List? getFullBuffer() {
    if (_position == buffer.length) {
      _position = 0;
      return Float32List.fromList(buffer);
    }
    return null;
  }
}
```

## 测试

### 单元测试

```dart
// test/sherpa_onnx_test.dart
import 'package:test/test.dart';
import 'package:sherpa_onnx/sherpa_onnx.dart';

void main() {
  group('Sherpa-ONNX Tests', () {
    test('创建识别器', () {
      final modelConfig = OfflineModelConfig(
        transducer: OfflineTransducerModelConfig(
          encoder: 'test_models/encoder.onnx',
          decoder: 'test_models/decoder.onnx',
          joiner: 'test_models/joiner.onnx',
        ),
        tokens: 'test_models/tokens.txt',
      );

      final recognizer = OfflineRecognizer(modelConfig);
      expect(recognizer, isNotNull);
      recognizer.dispose();
    });

    test('识别音频', () async {
      // 测试音频识别
    });
  });
}
```

## 部署

### 1. Web 部署

```dart
// 使用 dart2js 编译
dart compile js bin/asr.dart -o asr.js

// 创建 HTML
/*
<!DOCTYPE html>
<html>
<head>
    <title>Sherpa-ONNX Web Demo</title>
</head>
<body>
    <script type="text/javascript" src="asr.js"></script>
</body>
</html>
*/
```

### 2. 服务器部署

```yaml
# 使用 Docker
FROM google/dart:stable

WORKDIR /app
COPY . .
RUN dart pub get
RUN dart compile exe bin/server.dart -o server

EXPOSE 8080
CMD ["./server"]
```

## 故障排除

### 1. 模型加载失败
```dart
try {
  final recognizer = OfflineRecognizer(modelConfig);
} catch (e) {
  print('模型加载失败: $e');
  // 检查文件路径和格式
}
```

### 2. 音频格式问题
```dart
// 验证音频格式
void validateAudioFormat(Float32List samples, int sampleRate) {
  if (sampleRate != 16000) {
    throw ArgumentError('只支持 16kHz 采样率');
  }
  if (samples.isEmpty) {
    throw ArgumentError('音频数据为空');
  }
}
```

## 相关资源

- [Dart 官方文档](https://dart.dev/guides)
- [Flutter 开发文档](https://flutter.dev/docs)
- [Sherpa-ONNX Dart 包](https://pub.dev/packages/sherpa_onnx)
- [Aqueduct 框架](https://aqueduct.io/)

---

*更多示例请参考 Flutter 示例目录：[../flutter-examples](../flutter-examples/CLAUDE.md)*