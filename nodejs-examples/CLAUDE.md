# Node.js 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / Node.js 示例

## 模块概述

本模块包含 Sherpa-ONNX Node.js 绑定的使用示例。这些示例展示了如何在 JavaScript/TypeScript 环境中使用 Sherpa-ONNX 进行语音处理，包括纯 JavaScript 实现和基于 WebAssembly 的版本。

## 功能分类

### 🎤 语音识别 (ASR) 示例

#### 离线识别
- **test-offline-transducer.js**: Transducer 模型离线识别
- **test-offline-paraformer.js**: Paraformer 模型离线识别
- **test-offline-paraformer-itn.js**: 带 ITN（逆文本标准化）的 Paraformer
- **test-offline-sense-voice.js**: SenseVoice 多语言识别
- **test-offline-sense-voice-with-hr.js**: SenseVoice 带热词支持
- **test-offline-nemo-ctc.js**: NVIDIA NeMo CTC 模型
- **test-offline-moonshine.js**: Moonshine 轻量级模型
- **test-offline-dolphin-ctc.js**: Dolphin CTT 模型
- **test-offline-fire-red-asr.js**: FireRed 中文 ASR
- **test-offline-omnilingual-asr-ctc.js**: Omnilingual 多语言 CTC
- **test-offline-nemo-canary.js**: NeMo Canary 模型

#### 关键词检测
- **test-keyword-spotter-transducer.js**: 基于 Transducer 的关键词检测

### 👥 说话人处理
- **test-offline-speaker-diarization.js**: 说话人分离
- **test-offline-speaker-embedding.js**: 说话人特征提取
- **test-offline-speaker-verification.js**: 说话人验证

### 🎯 其他功能
- **test-offline-speech-enhancement-gtcrn.js**: GTCRN 语音增强
- **test-offline-punctuation.js**: 标点符号恢复

## 快速开始

### 1. 环境要求
- Node.js 16.0 或更高版本
- npm 或 yarn

### 2. 安装依赖
```bash
npm install
# 或
yarn install
```

### 3. 运行示例
```bash
node test-offline-transducer.js
```

## 使用示例

### 基本离线识别

```javascript
// test-offline-transducer.js
const sherpa_onnx = require('sherpa-onnx-node');

// 配置模型
const modelConfig = {
  transducer: {
    encoder: './encoder.onnx',
    decoder: './decoder.onnx',
    joiner: './joiner.onnx',
  },
  tokens: './tokens.txt',
  numThreads: 2,
  modelType: 'zipformer',
};

// 创建识别器
const recognizer = new sherpa_onnx.OfflineRecognizer(modelConfig);

// 创建音频流
const stream = recognizer.createStream();

// 读取音频文件
const waveData = sherpa_onnx.readWave('./test.wav');
stream.acceptWaveform(waveData.samples);

// 执行识别
recognizer.decode(stream);
const result = recognizer.getResult(stream);

console.log('识别结果:', result.text);
```

### SenseVoice 多语言识别

```javascript
// test-offline-sense-voice.js
const sherpa_onnx = require('sherpa-onnx-node');

// 配置 SenseVoice 模型
const modelConfig = {
  tokens: './tokens.txt',
  senseVoice: {
    model: './model.onnx',
  },
  numThreads: 2,
  modelType: 'sensevoice',
};

const recognizer = new sherpa_onnx.OfflineRecognizer(modelConfig);

// 识别多语言音频
const stream = recognizer.createStream();
const waveData = sherpa_onnx.readWave('./multilingual.wav');

stream.acceptWaveform(waveData.samples);
recognizer.decode(stream);

const result = recognizer.getResult(stream);
console.log('文本:', result.text);
console.log('语言:', result.lang); // 'zh', 'en', 'ja', 'ko', 'yue'
```

### 带热词的识别

```javascript
// test-offline-sense-voice-with-hr.js
const sherpa_onnx = require('sherpa-onnx-node');

// 配置热词
const hotwordsFile = './hotwords.txt';
const hotwords = [
  'sherpa-onnx',
  '语音识别',
  'Speech Recognition'
];

// 写入热词文件
const fs = require('fs');
fs.writeFileSync(hotwordsFile, hotwords.join('\n'));

// 配置模型
const modelConfig = {
  tokens: './tokens.txt',
  senseVoice: {
    model: './model.onnx',
  },
  hotwordsFile: hotwordsFile,
  hotwordsScore: 1.5,
};

const recognizer = new sherpa_onnx.OfflineRecognizer(modelConfig);

// 识别过程...
```

### 关键词检测

```javascript
// test-keyword-spotter-transducer.js
const sherpa_onnx = require('sherpa-onnx-node');

// 配置关键词
const keywordsConfig = {
  model: {
    transducer: {
      encoder: './encoder.onnx',
      decoder: './decoder.onnx',
      joiner: './joiner.onnx',
    },
    tokens: './tokens.txt',
  },
  keywords: {
    'hello world': 0.5,
    '你好': 0.5,
  },
  keywordsThreshold: 0.3,
};

const spotter = new sherpa_onnx.KeywordSpotter(keywordsConfig);
const stream = spotter.createStream();

// 处理音频
const audioData = readAudioData();
stream.acceptWaveform(audioData);
spotter.decode(stream);

const result = spotter.getResult(stream);
if (result.keyword) {
  console.log(`检测到关键词: ${result.keyword} (置信度: ${result.prob})`);
}
```

### 说话人分离

```javascript
// test-offline-speaker-diarization.js
const sherpa_onnx = require('sherpa-onnx-node');

// 配置说话人分离模型
const segmentationConfig = {
  sherpaDiarization: {
    model: './seg_model.onnx',
  },
  numThreads: 2,
};

const embeddingConfig = {
  speakerEmbedding: {
    model: './embedding_model.onnx',
  },
  numThreads: 2,
};

// 创建分离器
const segmenter = new sherpa_onnx.SegmentationModel(segmentationConfig);
const extractor = new sherpa_onnx.SpeakerEmbeddingExtractor(embeddingConfig);

const diarizationConfig = {
  segmentation: segmenter,
  embedding: extractor,
  clustering: {
    numClusters: 2,  // 说话人数量
    threshold: 0.5, // 聚类阈值
  },
};

const diarization = new sherpa_onnx.SpeakerDiarization(diarizationConfig);

// 处理音频
const waveData = sherpa_onnx.readWave('./meeting.wav');
const segments = diarization.process(waveData.samples, waveData.sampleRate);

// 输出结果
segments.forEach((segment, i) => {
  console.log(`段 ${i}: 说话人 ${segment.speaker} (${segment.start}s - ${segment.end}s)`);
});
```

## 实时音频处理

### 使用麦克风输入

```javascript
// real-time-asr.js
const sherpa_onnx = require('sherpa-onnx-node');
const { record } = require('node-record-lpcm16');

// 配置实时识别
const modelConfig = {
  tokens: './tokens.txt',
  transducer: {
    encoder: './encoder.onnx',
    decoder: './decoder.onnx',
    joiner: './joiner.onnx',
  },
  numThreads: 2,
};

const recognizerConfig = {
  modelConfig: modelConfig,
  enableEndpoint: true,
  rule1MinSilenceDuration: 0.5,
};

const recognizer = new sherpa_onnx.OnlineRecognizer(recognizerConfig);

// 创建录音流
const recording = record({
  threshold: 0.5,      // 静音阈值
  verbose: false,      // 不输出日志
  recordProgram: 'rec', // Linux/macOS 使用 rec
  silence: '1.0',      // 1秒静音后停止
});

recording.stream().on('data', (chunk) => {
  // 转换音频格式
  const samples = new Float32Array(chunk.length / 2);
  for (let i = 0; i < samples.length; i++) {
    samples[i] = chunk.readInt16LE(i * 2) / 32768.0;
  }

  // 创建流并识别
  const stream = recognizer.createStream();
  stream.acceptWaveform(samples);
  recognizer.decode(stream);

  const result = recognizer.getResult(stream);
  if (result.text) {
    console.log('实时识别:', result.text);
  }
});

recording.start();
```

## TypeScript 支持

### 安装类型定义
```bash
npm install --save-dev @types/sherpa-onnx-node
```

### TypeScript 示例
```typescript
// asr.ts
import * as sherpa_onnx from 'sherpa-onnx-node';

interface ModelConfig {
  transducer: {
    encoder: string;
    decoder: string;
    joiner: string;
  };
  tokens: string;
  numThreads: number;
  modelType: string;
}

const modelConfig: ModelConfig = {
  transducer: {
    encoder: './encoder.onnx',
    decoder: './decoder.onnx',
    joiner: './joiner.onnx',
  },
  tokens: './tokens.txt',
  numThreads: 2,
  modelType: 'zipformer',
};

const recognizer = new sherpa_onnx.OfflineRecognizer(modelConfig);
// ...
```

## 性能优化

### 1. Worker 线程
使用 Worker 进行后台处理：
```javascript
// worker.js
const { parentPort, workerData } = require('worker_threads');
const sherpa_onnx = require('sherpa-onnx-node');

const recognizer = new sherpa_onnx.OfflineRecognizer(workerData.config);

parentPort.on('message', (audioData) => {
  const stream = recognizer.createStream();
  stream.acceptWaveform(audioData);
  recognizer.decode(stream);
  const result = recognizer.getResult(stream);
  parentPort.postMessage(result);
});

// main.js
const { Worker } = require('worker_threads');

const worker = new Worker('./worker.js', {
  workerData: { config: modelConfig }
});

worker.on('message', (result) => {
  console.log('识别结果:', result.text);
});
```

### 2. 音频缓冲优化
```javascript
class AudioBuffer {
  constructor(size = 1600) {
    this.buffer = new Float32Array(size);
    this.index = 0;
  }

  push(data) {
    for (let i = 0; i < data.length; i++) {
      this.buffer[this.index++] = data[i];
      if (this.index >= this.buffer.length) {
        this.flush();
      }
    }
  }

  flush() {
    if (this.index > 0) {
      const chunk = this.buffer.slice(0, this.index);
      this.process(chunk);
      this.index = 0;
    }
  }

  process(chunk) {
    // 处理音频数据
  }
}
```

## 部署

### 1. Web 集成
```javascript
// 在 Express 服务器中
const express = require('express');
const multer = require('multer');
const sherpa_onnx = require('sherpa-onnx-node');

const app = express();
const upload = multer({ dest: 'uploads/' });

app.post('/asr', upload.single('audio'), async (req, res) => {
  const recognizer = new sherpa_onnx.OfflineRecognizer(modelConfig);
  const stream = recognizer.createStream();

  // 处理上传的音频文件
  const waveData = sherpa_onnx.readWave(req.file.path);
  stream.acceptWaveform(waveData.samples);
  recognizer.decode(stream);

  const result = recognizer.getResult(stream);
  res.json({ text: result.text });
});

app.listen(3000);
```

### 2. Serverless 部署
```javascript
// AWS Lambda 函数
const sherpa_onnx = require('sherpa-onnx-node');

let recognizer;

exports.handler = async (event) => {
  // 懒加载识别器
  if (!recognizer) {
    recognizer = new sherpa_onnx.OfflineRecognizer(modelConfig);
  }

  // 处理音频
  const stream = recognizer.createStream();
  // ... 处理逻辑 ...

  return {
    statusCode: 200,
    body: JSON.stringify({ text: result.text })
  };
};
```

## 故障排除

### 1. 模块加载失败
```bash
# 清除 npm 缓存
npm cache clean --force

# 重新安装
npm install
```

### 2. 音频文件格式错误
确保音频格式正确：
```javascript
// 验证 WAV 文件
const waveData = sherpa_onnx.readWave('./test.wav');
console.log('采样率:', waveData.sampleRate);
console.log('通道数:', waveData.numChannels);
console.log('样本数:', waveData.samples.length);
```

### 3. 内存使用过高
定期清理：
```javascript
setInterval(() => {
  if (global.gc) {
    global.gc(); // 手动触发垃圾回收
  }
}, 30000);
```

## 相关文档

- [Node.js 官方文档](https://nodejs.org/docs/)
- [Sherpa-ONNX Node.js 绑定](https://github.com/k2-fsa/sherpa-onnx/tree/master/nodejs-examples)
- [WebAssembly 集成](../wasm/CLAUDE.md)
- [Node.js Addon 示例](../nodejs-addon-examples/CLAUDE.md)

---

*更多示例和详细信息请参考项目 README 和官方文档。*