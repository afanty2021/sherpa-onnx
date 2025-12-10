[根目录](../CLAUDE.md) > **wasm**

# WebAssembly 支持

> 更新时间：2025-12-10 07:48:28

## 模块职责

Sherpa-ONNX 的 WebAssembly 模块将强大的语音 AI 功能带入浏览器和 Node.js 环境，实现了纯前端的语音处理能力。该模块通过 Emscripten 将 C++ 代码编译为 WebAssembly，提供了与原生版本几乎相同的性能，同时支持：

- **浏览器端**：无需后端服务器的语音处理
- **Node.js**：服务端的 JavaScript/TypeScript 集成
- **全功能支持**：ASR、TTS、VAD、说话人识别等
- **实时处理**：低延迟的流式处理

## 目录结构

```
wasm/
├── asr/                     # 语音识别
│   ├── index.html          # 浏览器演示
│   ├── sherpa-onnx-asr.js  # JavaScript API
│   └── sherpa-onnx-wasm-main-asr.cc
├── tts/                     # 语音合成
│   ├── index.html
│   ├── sherpa-onnx-tts.js
│   └── sherpa-onnx-wasm-main-tts.cc
├── vad/                     # 语音活动检测
│   ├── index.html
│   ├── sherpa-onnx-vad.js
│   └── sherpa-onnx-wasm-main-vad.cc
├── kws/                     # 关键词检测
├── speaker-diarization/     # 说话人日志
├── speech-enhancement/      # 语音增强
├── vad-asr/                 # VAD + ASR 组合
└── nodejs/                  # Node.js 支持
    ├── sherpa-onnx-wasm-nodejs.cc
    └── sherpa-onnx-wave.js
```

## 入口与启动

### 浏览器快速开始

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://cdn.jsdelivr.net/npm/onnxruntime-web/dist/ort.min.js"></script>
    <script src="./sherpa-onnx-asr.js"></script>
</head>
<body>
    <button id="start">开始识别</button>
    <button id="stop">停止识别</button>
    <div id="result"></div>

    <script>
        // 初始化识别器
        const recognizer = new SherpaOnnx.Asr({
            modelConfig: {
                zipformer: {
                    encoder: "./encoder-0.onnx",
                    decoder: "./decoder-0.onnx",
                    joiner: "./joiner-0.onnx"
                },
                tokens: "./tokens.txt"
            },
            sampleRate: 16000
        });

        // 获取麦克风权限
        navigator.mediaDevices.getUserMedia({ audio: true })
            .then(stream => {
                // 处理音频流...
            });
    </script>
</body>
</html>
```

### Node.js 快速开始

```javascript
const { createRequire } = require('module');
const require = createRequire(import.meta.url);
const sherpa_onnx = require('sherpa-onnx-wasm-nodejs');

// 创建识别器
const recognizerConfig = {
    modelConfig: {
        zipformer: {
            encoder: './encoder.onnx',
            decoder: './decoder.onnx',
            joiner: './joiner.onnx'
        },
        tokens: './tokens.txt',
        numThreads: 2
    },
    sampleRate: 16000
};

const recognizer = new sherpa_onnx.OfflineRecognizer(recognizerConfig);

// 识别音频文件
const audio = sherpa_onnx.readWave('test.wav');
const stream = recognizer.createStream();
stream.acceptWaveform(audio.sampleRate, audio.samples);

recognizer.decodeStream(stream);
const result = stream.result;
console.log('识别结果:', result.text);
```

## 对外接口

### 1. 浏览器 API

#### 语音识别 (ASR)
```javascript
// 创建在线识别器
const recognizer = new SherpaOnnx.Asr({
    // 配置
    modelConfig: {
        // Zipformer 模型
        zipformer: {
            encoder: './zipformer-encoder.onnx',
            decoder: './zipformer-decoder.onnx',
            joiner: './zipformer-joiner.onnx'
        },
        // 或 Paraformer 模型
        paraformer: {
            model: './paraformer.onnx'
        },
        // 或 Whisper 模型
        whisper: {
            encoder: './whisper-encoder.onnx',
            decoder: './whisper-decoder.onnx'
        },
        tokens: './tokens.txt',
        numThreads: 2,
        debug: false
    },
    featConfig: {
        sampleRate: 16000,
        featureDim: 80
    },
    decodingMethod: 'greedy_search'
});

// 创建流
const stream = recognizer.createStream();

// 处理音频
function processAudio(audioData) {
    stream.acceptWaveform(16000, audioData);

    if (recognizer.isReady(stream)) {
        recognizer.decodeStream(stream);
        const result = recognizer.getResult(stream);
        if (result.text) {
            console.log('识别结果:', result.text);
        }
    }
}
```

#### 语音合成 (TTS)
```javascript
// 创建 TTS
const tts = new SherpaOnnx.Tts({
    model: {
        vits: {
            model: './vits.onnx',
            lexicon: './lexicon.txt'
        },
        tokens: './tokens.txt'
    },
    ruleFsts: './rule.fst',
    maxNumSentences: 1
});

// 生成语音
const audio = tts.generate('Hello World!', {
    speakerId: 0,
    speed: 1.0
});

// 播放音频
const audioContext = new AudioContext();
const buffer = audioContext.createBuffer(
    1,
    audio.samples.length,
    audio.sampleRate
);
buffer.getChannelData(0).set(audio.samples);
const source = audioContext.createBufferSource();
source.buffer = buffer;
source.connect(audioContext.destination);
source.start();
```

### 2. Node.js API

#### C 绑定接口
```javascript
const sherpa_onnx = require('sherpa-onnx-wasm-nodejs');

// 离线识别器
class OfflineRecognizer {
    constructor(config);
    createStream();
    decodeStream(stream);
    getResult(stream);
}

// 在线识别器
class OnlineRecognizer {
    constructor(config);
    createStream();
    isReady(stream);
    decodeStream(stream);
    getResult(stream);
    reset(stream);
}

// TTS
class OfflineTts {
    constructor(config);
    generate(text, options);
}

// 说话人识别
class SpeakerEmbeddingExtractor {
    constructor(config);
    createStream();
    compute(stream);
}
```

## 关键依赖与配置

### 浏览器依赖
- **onnxruntime-web**: ONNX 模型推理
- **Web Audio API**: 音频捕获和播放
- **MediaDevices API**: 麦克风访问
- **Web Workers**: 并行处理（可选）

### Node.js 依赖
- **onnxruntime-node**: Node.js 版 ONNX Runtime
- **node-wav**: WAV 文件读写
- **speaker**: 音频播放（可选）

### 构建配置

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.13)
project(sherpa-onnx-wasm)

# 设置 Emscripten 工具链
set(CMAKE_TOOLCHAIN_FILE ${EMSDK}/upstream/emscripten/cmake/Modules/Platform/Emscripten.cmake)

# 配置选项
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -flto")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -flto")

# 启用异常
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fexceptions")

# 导出函数
set(CMAKE_EXECUTABLE_LINKER_FLAGS
    "${CMAKE_EXECUTABLE_LINKER_FLAGS} -s EXPORTED_FUNCTIONS=['_main'] -s EXPORTED_RUNTIME_METHODS=['ccall', 'cwrap']")

# 优化内存
set(CMAKE_EXECUTABLE_LINKER_FLAGS
    "${CMAKE_EXECUTABLE_LINKER_FLAGS} -s ALLOW_MEMORY_GROWTH=1 -s MAXIMUM_MEMORY=4GB")

# Node.js 支持
if(NOT BUILD_FOR_BROWSER)
    set(CMAKE_EXECUTABLE_LINKER_FLAGS
        "${CMAKE_EXECUTABLE_LINKER_FLAGS} -s WASM=1 -s EXPORT_ES6=1")
endif()
```

## 性能优化

### 1. 模型优化
```javascript
// 使用量化模型
modelConfig: {
    model: './model-quantized.onnx'  // INT8 量化，更小更快
}

// 模型分块加载（大模型）
const chunkPromises = [
    fetch('encoder_chunk1.onnx'),
    fetch('encoder_chunk2.onnx'),
    // ...
];
```

### 2. 内存管理
```javascript
// 及时释放资源
function cleanup() {
    stream.free();
    recognizer.free();
}

// 使用对象池管理音频缓冲区
class BufferPool {
    constructor(size = 10, bufferSize = 4096) {
        this.pool = Array(size).fill().map(() => new Float32Array(bufferSize));
        this.index = 0;
    }

    acquire() {
        return this.pool[this.index++ % this.pool.length];
    }
}
```

### 3. Web Workers 并行
```javascript
// 主线程
const worker = new Worker('asr-worker.js');
worker.postMessage({
    type: 'init',
    config: recognizerConfig
});

// worker.js
self.onmessage = function(e) {
    if (e.data.type === 'init') {
        // 在 Worker 中初始化识别器
        recognizer = new SherpaOnnx.Asr(e.data.config);
    } else if (e.data.type === 'audio') {
        // 处理音频
        const result = processAudio(e.data.audio);
        self.postMessage({ type: 'result', text: result.text });
    }
};
```

## 部署指南

### 1. 静态网站部署
```bash
# 构建生产版本
cd wasm/asr
emmake make build

# 部署到 CDN
aws s3 sync ./dist s3://your-bucket/sherpa-onnx --acl public-read
```

### 2. Node.js 包发布
```bash
# 构建 Node.js 模块
cd wasm/nodejs
emcmake cmake ..
emmake make

# 打包发布
npm pack
npm publish
```

### 3. Docker 部署
```dockerfile
FROM node:18-alpine

# 安装 Emscripten
RUN wget -q https://github.com/emscripten-core/emsdk/archive/main.tar.gz
RUN tar -xzf main.tar.gz
RUN cd emsdk-main && ./emsdk install latest && ./emsdk activate latest

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build:wasm

EXPOSE 3000
CMD ["npm", "start"]
```

## 测试与质量

### 单元测试
```javascript
// 使用 Jest 测试
describe('Sherpa-ONNX WASM', () => {
    test('ASR 基础功能', async () => {
        const recognizer = new SherpaOnnx.Asr(config);
        const stream = recognizer.createStream();

        stream.acceptWaveform(16000, testAudio);
        recognizer.decodeStream(stream);
        const result = recognizer.getResult(stream);

        expect(result.text).toBe('hello world');
    });
});
```

### 性能基准测试
```javascript
function benchmarkASR(recognizer, audioData) {
    const startTime = performance.now();

    const stream = recognizer.createStream();
    stream.acceptWaveform(16000, audioData);
    recognizer.decodeStream(stream);

    const endTime = performance.now();
    const duration = (endTime - startTime) / 1000;
    const audioDuration = audioData.length / 16000;
    const rtf = duration / audioDuration;

    console.log(`RTF: ${rtf.toFixed(2)}`);
    return rtf;
}
```

## 常见问题 (FAQ)

### Q1: WebAssembly 加载失败？
A: 检查：
- MIME 类型配置（.wasm 应为 application/wasm）
- CORS 设置
- 文件路径正确性

### Q2: 浏览器内存不足？
A: 优化方案：
```javascript
// 增加内存限制
var Module = {
    TOTAL_MEMORY: 1073741824,  // 1GB
    ALLOW_MEMORY_GROWTH: 1
};

// 或使用 SharedArrayBuffer（需要 COOP/COEP 头）
```

### Q3: 音频延迟过高？
A: 解决方案：
- 减小缓冲区大小
- 使用 WebAudio API 的 scriptProcessor
- 启用硬件加速

### Q4: 模型加载慢？
A: 优化建议：
- 使用 CDN
- 启用 gzip/brotli 压缩
- 预加载模型文件

### Q5: Node.js 版本兼容性？
A: 需求：
- Node.js 14+
- 支持 WASM 的环境
- onnxruntime-node 版本匹配

## 相关文件清单

### 核心文件
- `sherpa-onnx-wasm-main-asr.cc` - ASR 主入口
- `sherpa-onnx-wasm-main-tts.cc` - TTS 主入口
- `sherpa-onnx-wasm-nodejs.cc` - Node.js 绑定

### JavaScript API
- `asr/sherpa-onnx-asr.js` - ASR API
- `tts/sherpa-onnx-tts.js` - TTS API
- `vad/sherpa-onnx-vad.js` - VAD API

### 演示页面
- `asr/index.html` - ASR 演示
- `tts/index.html` - TTS 演示
- `vad-asr/index.html` - VAD+ASR 组合演示

### 构建脚本
- `CMakeLists.txt` - 构建配置
- `build.sh` - 构建脚本

## 变更记录

### 2025-12-10 07:48:28
- 📝 创建 WebAssembly 文档
- 🌐 添加浏览器和 Node.js 使用示例
- ⚡ 补充性能优化建议
- 🔧 记录部署和故障排除指南

---

*相关文件：[C++ 核心](../sherpa-onnx/csrc/CLAUDE.md) | [Node.js 集成](./nodejs/README.md) | [构建系统](../build.sh)*