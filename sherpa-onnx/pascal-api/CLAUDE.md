# Pascal API 绑定

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / Pascal API

## 模块概述

本模块提供 Sherpa-ONNX 的 Object Pascal 语言绑定，使 Pascal 开发者能够在他们的应用中集成离线语音处理功能。它包含了完整的 Pascal API 定义和 PortAudio 音频库的 Pascal 绑定。

## 文件结构

```
sherpa-onnx/pascal-api/
├── sherpa_onnx.pas    # 主要的 Sherpa-ONNX API 定义
├── portaudio.pas      # PortAudio 音频库的 Pascal 绑定
└── README.md          # 简要说明
```

## 主要组件

### sherpa_onnx.pas
核心 API 定义文件，包含了：

#### 1. 核心结构体
- `SherpaOnnxOnlineModelConfig`: 在线模型配置
- `SherpaOnnxOfflineModelConfig`: 离线模型配置
- `SherpaOnnxOnlineRecognizerConfig`: 在线识别器配置
- `SherpaOnnxOfflineRecognizerConfig`: 离线识别器配置

#### 2. 识别器类
- `SherpaOnnxOnlineRecognizer`: 实时语音识别
- `SherpaOnnxOfflineRecognizer`: 文件语音识别
- `SherpaOnnxKeywordSpotter`: 关键词检测
- `SherpaOnnxVadModel`: 语音活动检测
- `SherpaOnnxSpeakerEmbeddingExtractor`: 说话人特征提取

#### 3. 语音合成
- `SherpaOnnxOfflineTtsModel`: 离线 TTS 模型
- `SherpaOnnxOfflineTts`: 语音合成器
- `SherpaOnnxGeneratedAudio`: 生成的音频数据

#### 4. 音频处理
- `SherpaOnnxSpeechEnhancementModel`: 语音增强模型
- `SherpaOnnxSpeechEnhancement`: 语音增强器

### portaudio.pas
PortAudio 跨平台音频 I/O 库的 Pascal 绑定，提供：
- 音频设备枚举和管理
- 音频流创建和控制
- 实时音频输入输出
- 回调函数接口

## 使用示例

### 基本语音识别

```pascal
program AsrDemo;

{$mode objfpc}{$H+}

uses
  Classes, SysUtils, sherpa_onnx;

var
  ModelConfig: SherpaOnnxOnlineModelConfig;
  RecognizerConfig: SherpaOnnxOnlineRecognizerConfig;
  Recognizer: SherpaOnnxOnlineRecognizer;
  Stream: SherpaOnnxOnlineStream;
  Samples: array of Single;
  Result: string;
begin
  // 配置模型
  ModelConfig.Transducer.Encoder := 'encoder.onnx';
  ModelConfig.Transducer.Decoder := 'decoder.onnx';
  ModelConfig.Transducer.Joiner := 'joiner.onnx';
  ModelConfig.Tokens := 'tokens.txt';
  ModelConfig.NumThreads := 1;
  ModelConfig.ModelType := 'zipformer';

  // 配置识别器
  RecognizerConfig.ModelConfig := ModelConfig;
  RecognizerConfig.EnableEndpoint := True;

  // 创建识别器
  Recognizer := SherpaOnnxOnlineRecognizer.Create(RecognizerConfig);

  // 创建音频流
  Stream := Recognizer.CreateStream;

  // 模拟音频输入
  SetLength(Samples, 1600); // 100ms @ 16kHz
  // ... 填充音频数据 ...

  // 识别
  Stream.AcceptWaveform(16000, Samples, Length(Samples));
  Recognizer.Decode(Stream);

  // 获取结果
  Result := Recognizer.GetResult(Stream);
  WriteLn('识别结果: ', Result);

  // 清理
  Stream.Destroy;
  Recognizer.Destroy;
end.
```

### 语音合成

```pascal
program TtsDemo;

{$mode objfpc}{$H+}

uses
  Classes, SysUtils, sherpa_onnx;

var
  ModelConfig: SherpaOnnxOfflineTtsModelConfig;
  Model: SherpaOnnxOfflineTtsModel;
  Tts: SherpaOnnxOfflineTts;
  Audio: SherpaOnnxGeneratedAudio;
begin
  // 配置 TTS 模型
  ModelConfig.Vits.Model := 'vits.onnx';
  ModelConfig.Vits.Lexicon := 'lexicon.txt';
  ModelConfig.Vits.Tokens := 'tokens.txt';
  ModelConfig.NumThreads := 1;

  // 创建模型和合成器
  Model := SherpaOnnxOfflineTtsModel.Create(ModelConfig);
  Tts := SherpaOnnxOfflineTts.Create(Model);

  // 生成语音
  Audio := Tts.Generate('Hello, world!', 0); // 说话人 ID = 0

  // 保存音频
  with TFileStream.Create('output.wav', fmCreate) do
  begin
    WriteBuffer(Audio.WavData[0], Length(Audio.WavData));
    Free;
  end;

  // 清理
  Audio.Destroy;
  Tts.Destroy;
  Model.Destroy;
end.
```

## 编译要求

### Free Pascal / Lazarus
- Free Pascal Compiler 3.0.4+
- Lazarus IDE (可选)

### 编译命令
```bash
# 使用 fpc 直接编译
fpc -Mobjfpc -Scghi -O1 -g -gl -l -vewnhibq your_program.pas

# 使用 Lazarus IDE
1. 打开 Lazarus
2. 创建新项目 (Project > New Project)
3. 添加 sherpa_onnx.pas 到项目路径
4. 编译运行
```

### 依赖库
- ONNX Runtime (动态链接)
- PortAudio (可选，用于音频 I/O)

## 平台支持

### Windows
- 需要 sherpa_onnx.dll
- 需要 portaudio.dll (如果使用 PortAudio)

### Linux
- 需要 libsherpa-onnx.so
- 需要 libportaudio.so (如果使用 PortAudio)

### macOS
- 需要 libsherpa-onnx.dylib
- 需要 libportaudio.dylib (如果使用 PortAudio)

## 相关文档

- [Pascal API 示例](../../pascal-api-examples/CLAUDE.md)
- [Object Pascal 官方文档](https://www.freepascal.org/docs.html)
- [Lazarus IDE](https://www.lazarus-ide.org/)
- [Sherpa-ONNX 核心模块](../csrc/CLAUDE.md)

---

*更多示例和详细信息请参考：https://github.com/k2-fsa/sherpa-onnx/tree/master/pascal-api-examples*