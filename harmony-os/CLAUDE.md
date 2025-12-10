[根目录](../../CLAUDE.md) > **harmony-os**

# HarmonyOS 支持

> 最后更新：2025-12-10 07:52:21

## 模块职责

HarmonyOS 模块为华为鸿蒙系统提供 sherpa-onnx 的完整支持，包括：

- **HAR 包构建**：提供预编译的 sherpa_onnx.har 包供开发者直接使用
- **示例应用**：包含完整的示例应用展示各种语音功能
- **原生绑定**：通过 NAPI 技术实现 C++ 核心与 ArkTS 的无缝集成
- **多架构支持**：支持 arm64-v8a、armeabi-v7a、x86_64 等架构

## 入口与启动

### 项目结构

```
harmony-os/
├── SherpaOnnxHar/              # HAR 包构建项目
│   ├── sherpa_onnx/            # 核心模块源码
│   │   ├── src/main/cpp/       # C++ 实现
│   │   └── src/main/ets/       # ArkTS 接口
│   └── entry/                  # 测试应用
├── SherpaOnnxStreamingAsr/     # 流式语音识别示例
├── SherpaOnnxVadAsr/           # VAD+ASR 示例
├── SherpaOnnxTts/              # 语音合成示例
├── SherpaOnnxSpeakerDiarization/  # 说话人分离示例
└── SherpaOnnxSpeakerIdentification/ # 说话人识别示例
```

### 构建要求

- **DevEco Studio**: 4.0+
- **HarmonyOS API**: 9+
- **Node.js**: 16.0+
- **ohpm**: 包管理工具

## 对外接口

### ArkTS API 接口

#### 1. 流式语音识别 (Streaming ASR)

```typescript
// 创建在线识别器
const recognizer = createOnlineRecognizer(config);
const stream = createOnlineStream(recognizer);

// 接收音频数据
stream.acceptWaveform({
  samples: audioData,
  sampleRate: 16000
});

// 获取识别结果
const result = decodeOnlineStream(recognizer, stream);
```

#### 2. 非流式语音识别 (Offline ASR)

```typescript
// 创建离线识别器
const recognizer = createOfflineRecognizer(config);

// 识别整个音频文件
const result = recognizer.decode(audioData);
```

#### 3. 语音合成 (TTS)

```typescript
// 创建 TTS 实例
const tts = createOfflineTts(config);

// 生成语音
const audio = tts.generate("Hello, World!");
```

#### 4. 说话人识别

```typescript
// 说话人分离
const diarization = createOfflineSpeakerDiarization(config);
const segments = diarization.process(audioData);

// 说话人识别
const speakerId = identifySpeaker(speakerModel, audioData);
```

#### 5. 语音活动检测 (VAD)

```typescript
const vad = createVad(config);
const isSpeech = vad.process(audioChunk);
```

### 核心配置类

```typescript
// 在线模型配置
class OnlineModelConfig {
  transducer: OnlineTransducerModelConfig;
  paraformer: OnlineParaformerModelConfig;
  zipformer2Ctc: OnlineZipformer2CtcModelConfig;
  tokens: string;
  numThreads: number;
  provider: string;  // cpu, gpu, npu
  modelType: string;
}

// TTS 模型配置
class OfflineTtsModelConfig {
  vits: OfflineTtsVitsModelConfig;
  matchaTts: OfflineTtsMatchaModelConfig;
  kokoro: OfflineTtsKokoroModelConfig;
  numThreads: number;
  debug: boolean;
  provider: string;
}
```

## 关键依赖与配置

### 1. 原生依赖

- **ONNX Runtime**: 用于模型推理
- **sherpa-onnx 核心库**: C++ 实现的核心功能
- **NAPI**: Node.js API 用于原生交互

### 2. 包依赖 (oh-package.json5)

```json
{
  "dependencies": {
    "@ohos/napi": "^1.0.0",
    "libsherpa_onnx.so": "1.12.19"
  }
}
```

### 3. 构建配置 (build-profile.json5)

```json
{
  "app": {
    "signingConfigs": [],
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compileSdkVersion": 9,
        "compatibleSdkVersion": 9,
        "runtimeOS": "HarmonyOS"
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [
        {
          "name": "default",
          "applyToProducts": ["default"]
        }
      ]
    }
  ]
}
```

## 测试与质量

### 测试结构

```
src/ohosTest/ets/test/
├── Ability.test.ets      # 能力测试
└── List.test.ets         # 单元测试
```

### 测试覆盖

- ✅ 基础 API 调用测试
- ✅ 音频格式处理测试
- ⚠️ 性能基准测试（待补充）
- ⚠️ 内存泄漏测试（待补充）

## 常见问题 (FAQ)

### Q1: 如何处理不同架构的 so 库？

A: SherpaOnnxHar 项目已包含多架构支持，构建时自动选择对应架构：
- arm64-v8a: 主流 64 位 ARM 设备
- armeabi-v7a: 兼容旧款 32 位 ARM 设备
- x86_64: 模拟器和部分平板

### Q2: 模型文件如何管理？

A: 建议将模型文件放在 resources/raw/ 目录下，通过 $r 资源引用：

```typescript
const modelPath = this.context.resourceDir + '/raw/model/';
```

### Q3: 如何优化性能？

A: 性能优化建议：
1. 使用 NPU 加速（provider: "npu"）
2. 调整线程数（numThreads: 4）
3. 启用模型量化
4. 使用流式处理减少延迟

### Q4: 权限配置？

A: 需要在 module.json5 中添加：

```json
{
  "requestPermissions": [
    {
      "name": "ohos.permission.MICROPHONE",
      "reason": "需要使用麦克风进行语音识别",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    }
  ]
}
```

## 相关文件清单

### 核心实现文件

- `SherpaOnnxHar/sherpa_onnx/src/main/ets/components/StreamingAsr.ets` - 流式 ASR 实现
- `SherpaOnnxHar/sherpa_onnx/src/main/ets/components/NonStreamingAsr.ets` - 非流式 ASR
- `SherpaOnnxHar/sherpa_onnx/src/main/ets/components/NonStreamingTts.ets` - TTS 实现
- `SherpaOnnxHar/sherpa_onnx/src/main/ets/components/Vad.ets` - VAD 实现
- `SherpaOnnxHar/sherpa_onnx/src/main/ets/components/SpeakerIdentification.ets` - 说话人识别

### 原生代码文件

- `SherpaOnnxHar/sherpa_onnx/src/main/cpp/streaming-asr.cc` - 流式 ASR 原生实现
- `SherpaOnnxHar/sherpa_onnx/src/main/cpp/non-streaming-asr.cc` - 非流式 ASR
- `SherpaOnnxHar/sherpa_onnx/src/main/cpp/non-streaming-tts.cc` - TTS 原生实现
- `SherpaOnnxHar/sherpa_onnx/src/main/cpp/vad.cc` - VAD 原生实现

### 配置文件

- `SherpaOnnxHar/sherpa_onnx/oh-package.json5` - 包配置
- `SherpaOnnxHar/sherpa_onnx/build-profile.json5` - 构建配置
- `SherpaOnnxHar/sherpa_onnx/src/main/cpp/CMakeLists.txt` - 原生编译配置

## 变更记录 (Changelog)

### 2025-12-10 07:52:21
- 📝 创建 HarmonyOS 模块文档
- 🔧 整理 API 接口说明
- 📋 添加常见问题解答
- 📊 完成文件清单整理

---

*注：本模块持续更新中，建议关注官方文档获取最新信息：https://k2-fsa.github.io/sherpa/onnx/harmony-os/*