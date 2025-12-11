# WebAssembly API

<cite>
**本文档引用文件**   
- [CLAUDE.md](file://wasm/CLAUDE.md)
- [CMakeLists.txt](file://wasm/CMakeLists.txt)
- [sherpa-onnx-asr.js](file://wasm/asr/sherpa-onnx-asr.js)
- [sherpa-onnx-tts.js](file://wasm/tts/sherpa-onnx-tts.js)
- [sherpa-onnx-vad.js](file://wasm/vad/sherpa-onnx-vad.js)
- [sherpa-onnx-kws.js](file://wasm/kws/sherpa-onnx-kws.js)
- [sherpa-onnx-wasm-nodejs.cc](file://wasm/nodejs/sherpa-onnx-wasm-nodejs.cc)
- [sherpa-onnx-wave.js](file://wasm/nodejs/sherpa-onnx-wave.js)
</cite>

## 目录
1. [简介](#简介)
2. [WASM模块加载](#wasm模块加载)
3. [JavaScript/TypeScript绑定接口](#javascripttypescript绑定接口)
4. [内存管理](#内存管理)
5. [线程模型](#线程模型)
6. [Web Audio API集成](#web-audio-api集成)
7. [离线工作线程使用](#离线工作线程使用)
8. [性能监控](#性能监控)
9. [前端框架集成示例](#前端框架集成示例)
10. [WebGL可视化](#webgl可视化)
11. [实时流处理](#实时流处理)
12. [PWA支持](#pwa支持)
13. [浏览器兼容性](#浏览器兼容性)
14. [CORS策略](#cors策略)
15. [安全沙箱限制](#安全沙箱限制)

## 简介

sherpa-onnx的WebAssembly模块将强大的语音AI功能带入浏览器和Node.js环境，实现了纯前端的语音处理能力。该模块通过Emscripten将C++代码编译为WebAssembly，提供了与原生版本几乎相同的性能，同时支持ASR（语音识别）、TTS（语音合成）、VAD（语音活动检测）、说话人识别等功能。

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L1-L15)

## WASM模块加载

WebAssembly模块的加载需要正确配置MIME类型和CORS策略。在浏览器中，可以通过script标签引入JavaScript API文件，然后初始化模块。

```javascript
// 浏览器中加载WASM模块
<script src="./sherpa-onnx-asr.js"></script>
```

在Node.js环境中，可以通过require引入模块：

```javascript
// Node.js中加载WASM模块
const sherpa_onnx = require('sherpa-onnx-wasm-nodejs');
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L43-L112)

## JavaScript/TypeScript绑定接口

### 语音识别 (ASR)

创建在线识别器：

```javascript
const recognizer = new SherpaOnnx.Asr({
    modelConfig: {
        zipformer: {
            encoder: './zipformer-encoder.onnx',
            decoder: './zipformer-decoder.onnx',
            joiner: './zipformer-joiner.onnx'
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
```

创建流并处理音频：

```javascript
const stream = recognizer.createStream();
stream.acceptWaveform(16000, audioData);

if (recognizer.isReady(stream)) {
    recognizer.decodeStream(stream);
    const result = recognizer.getResult(stream);
    if (result.text) {
        console.log('识别结果:', result.text);
    }
}
```

### 语音合成 (TTS)

创建TTS实例：

```javascript
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
```

生成语音：

```javascript
const audio = tts.generate('Hello World!', {
    speakerId: 0,
    speed: 1.0
});
```

### 语音活动检测 (VAD)

创建VAD实例：

```javascript
const vad = new SherpaOnnx.Vad({
    sileroVad: {
        model: './silero_vad.onnx',
        threshold: 0.50,
        minSilenceDuration: 0.50,
        minSpeechDuration: 0.25,
        maxSpeechDuration: 20,
        windowSize: 512,
    },
    sampleRate: 16000,
    numThreads: 1,
    provider: 'cpu',
    debug: 1,
    bufferSizeInSeconds: 30,
});
```

处理音频流：

```javascript
vad.acceptWaveform(samples);
if (vad.isDetected()) {
    const segment = vad.front();
    // 处理检测到的语音段
    vad.pop();
}
```

### 关键词检测 (KWS)

创建关键词检测器：

```javascript
const kws = new SherpaOnnx.Kws({
    featConfig: {
        samplingRate: 16000,
        featureDim: 80,
    },
    modelConfig: {
        transducer: {
            encoder: './encoder-epoch-12-avg-2-chunk-16-left-64.onnx',
            decoder: './decoder-epoch-12-avg-2-chunk-16-left-64.onnx',
            joiner: './joiner-epoch-12-avg-2-chunk-16-left-64.onnx',
        },
        tokens: './tokens.txt',
        provider: 'cpu',
        modelType: '',
        numThreads: 1,
        debug: 1,
        modelingUnit: 'cjkchar',
        bpeVocab: '',
    },
    maxActivePaths: 4,
    numTrailingBlanks: 1,
    keywordsScore: 1.0,
    keywordsThreshold: 0.25,
    keywords: 'x iǎo ài t óng x ué @小爱同学\n' +
        'j ūn g ē n iú b ī @军哥牛逼'
});
```

**Section sources**
- [sherpa-onnx-asr.js](file://wasm/asr/sherpa-onnx-asr.js#L1-L800)
- [sherpa-onnx-tts.js](file://wasm/tts/sherpa-onnx-tts.js#L1-L614)
- [sherpa-onnx-vad.js](file://wasm/vad/sherpa-onnx-vad.js#L1-L324)
- [sherpa-onnx-kws.js](file://wasm/kws/sherpa-onnx-kws.js#L1-L365)

## 内存管理

WASM模块的内存管理需要特别注意，及时释放资源以避免内存泄漏。

```javascript
// 及时释放资源
function cleanup() {
    stream.free();
    recognizer.free();
}
```

使用对象池管理音频缓冲区：

```javascript
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

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L302-L320)

## 线程模型

WASM模块支持多线程处理，可以通过配置numThreads参数来指定使用的线程数。

```javascript
const recognizerConfig = {
    modelConfig: {
        // ...
        numThreads: 2
    },
    // ...
};
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L140-L141)

## Web Audio API集成

WASM模块与Web Audio API集成，可以实现音频的捕获和播放。

```javascript
// 获取麦克风权限
navigator.mediaDevices.getUserMedia({ audio: true })
    .then(stream => {
        // 处理音频流...
    });
```

播放生成的音频：

```javascript
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

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L71-L75)
- [sherpa-onnx-tts.js](file://wasm/tts/sherpa-onnx-tts.js#L188-L198)

## 离线工作线程使用

使用Web Workers进行并行处理，避免阻塞主线程。

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

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L323-L342)

## 性能监控

监控ASR的实时因子（RTF）：

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

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L408-L424)

## 前端框架集成示例

### React集成

```javascript
import React, { useEffect, useRef } from 'react';

function ASRComponent() {
    const recognizerRef = useRef(null);

    useEffect(() => {
        // 初始化识别器
        const recognizer = new SherpaOnnx.Asr(config);
        recognizerRef.current = recognizer;

        return () => {
            // 清理资源
            if (recognizerRef.current) {
                recognizerRef.current.free();
            }
        };
    }, []);

    return (
        <div>
            <button onClick={startRecognition}>开始识别</button>
            <button onClick={stopRecognition}>停止识别</button>
        </div>
    );
}
```

### Vue集成

```javascript
<template>
  <div>
    <button @click="startRecognition">开始识别</button>
    <button @click="stopRecognition">停止识别</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      recognizer: null
    };
  },
  mounted() {
    // 初始化识别器
    this.recognizer = new SherpaOnnx.Asr(config);
  },
  beforeDestroy() {
    // 清理资源
    if (this.recognizer) {
      this.recognizer.free();
    }
  }
};
</script>
```

### Angular集成

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';

@Component({
  selector: 'app-asr',
  template: `
    <button (click)="startRecognition()">开始识别</button>
    <button (click)="stopRecognition()">停止识别</button>
  `
})
export class ASRComponent implements OnInit, OnDestroy {
  private recognizer: any;

  ngOnInit() {
    // 初始化识别器
    this.recognizer = new SherpaOnnx.Asr(config);
  }

  ngOnDestroy() {
    // 清理资源
    if (this.recognizer) {
      this.recognizer.free();
    }
  }
}
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L45-L78)

## WebGL可视化

使用WebGL进行音频波形可视化：

```javascript
// 创建WebGL上下文
const canvas = document.getElementById('canvas');
const gl = canvas.getContext('webgl');

// 绘制音频波形
function drawWaveform(samples) {
    // WebGL绘制逻辑
}
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L45-L78)

## 实时流处理

实时处理音频流：

```javascript
// 处理实时音频流
function processAudioStream(stream) {
    const reader = stream.getReader();
    
    reader.read().then(function processChunk({ done, value }) {
        if (done) {
            return;
        }
        
        // 处理音频数据块
        const audioData = value;
        stream.acceptWaveform(16000, audioData);
        
        if (recognizer.isReady(stream)) {
            recognizer.decodeStream(stream);
            const result = recognizer.getResult(stream);
            if (result.text) {
                console.log('识别结果:', result.text);
            }
        }
        
        // 继续读取下一个数据块
        return reader.read().then(processChunk);
    });
}
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L154-L164)

## PWA支持

将WASM模块集成到PWA应用中，实现离线语音处理功能。

```javascript
// 注册Service Worker
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js')
        .then(registration => {
            console.log('Service Worker registered');
        })
        .catch(error => {
            console.log('Service Worker registration failed:', error);
        });
}
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L347-L367)

## 浏览器兼容性

WASM模块需要现代浏览器支持，推荐使用最新版本的Chrome、Firefox、Safari和Edge。

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L429-L433)

## CORS策略

确保WASM文件和模型文件的CORS策略正确配置，避免跨域请求问题。

```javascript
// 服务器端CORS配置示例
app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
    next();
});
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L431-L433)

## 安全沙箱限制

WASM模块在浏览器的安全沙箱中运行，无法直接访问文件系统，需要通过File API或Fetch API加载模型文件。

```javascript
// 通过Fetch API加载模型文件
fetch('./model.onnx')
    .then(response => response.arrayBuffer())
    .then(buffer => {
        // 处理模型文件
    });
```

**Section sources**
- [CLAUDE.md](file://wasm/CLAUDE.md#L431-L433)