# Go API 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / Go API 示例

## 模块概述

本模块包含 Sherpa-ONNX Go API 的使用示例，展示了如何使用 Go 语言实现各种离线语音处理功能。Go 语言的高并发特性使其非常适合构建实时语音处理服务。

## 功能分类

### 🎤 语音识别 (ASR) 示例

#### 非流式识别
- **non-streaming-decode-files**: 批量文件语音识别
- **non-streaming-canary-decode-files**: NVIDIA NeMo Canary 模型
- **non-streaming-omnilingual-asr-ctc-decode-files**: Omnilingual 多语言 CTC

#### 流式识别
- **streaming-decode-files**: 流式文件识别
- **streaming-hlg-decoding**: 带 HLG 解码的流式识别
- **real-time-speech-recognition-from-microphone**: 实时麦克风识别

### 🔊 语音合成 (TTS) 示例
- **non-streaming-tts**: 离线文本转语音
- **offline-tts-play**: 语音合成并播放

### 👥 说话人处理示例
- **non-streaming-speaker-diarization**: 说话人分离
- **speaker-identification**: 说话人识别

### 🎯 其他功能示例

#### 语音活动检测 (VAD)
- **vad**: Silero VAD 语音活动检测
- **vad-asr-paraformer**: VAD + Paraformer 识别
- **vad-asr-whisper**: VAD + Whisper 识别
- **vad-speaker-identification**: VAD + 说话人识别
- **vad-spoken-language-identification**: VAD + 语言识别

#### 文本处理
- **add-punctuation**: 标点符号恢复

#### 音频处理
- **audio-tagging**: 音频事件检测
- **speech-enhancement-gtcrn**: GTCRN 语音增强

## 快速开始

### 1. 环境要求
- Go 1.18 或更高版本
- CGO 支持
- ONNX Runtime C 库

### 2. 安装 Go
```bash
# macOS
brew install go

# Linux (Ubuntu)
sudo apt-get install golang-go

# 下载安装
wget https://golang.org/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
```

### 3. 安装 sherpa-onnx Go 库
```bash
go get github.com/k2-fsa/sherpa-onnx-go
```

### 4. 构建示例
```bash
cd non-streaming-decode-files
go run main.go
```

## 使用示例

### 基本非流式识别

```go
// non-streaming-decode-files/main.go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/k2-fsa/sherpa-onnx-go"
)

func main() {
	// 配置模型
	modelConfig := sherpaOnnx.OfflineModelConfig{
		Tokens: "./tokens.txt",
		Transducer: &sherpaOnnx.OfflineTransducerModelConfig{
			Encoder: "./encoder.onnx",
			Decoder: "./decoder.onnx",
			Joiner:  "./joiner.onnx",
		},
		NumThreads: 2,
		ModelType:  "zipformer",
	}

	// 创建识别器
	recognizer := sherpaOnnx.NewOfflineRecognizer(modelConfig)
	defer recognizer.Release()

	// 识别音频文件
	audioFile := "./test.wav"
	stream := recognizer.NewStream()
	defer stream.Release()

	// 加载音频
	waveData := sherpaOnnx.ReadWave(audioFile)
	stream.AcceptWaveform(waveData.Samples)

	// 执行识别
	recognizer.Decode(stream)
	result := recognizer.Result(stream)

	fmt.Printf("识别结果: %s\n", result.Text)
}
```

### 实时麦克风识别

```go
// real-time-speech-recognition-from-microphone/main.go
package main

import (
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"

	"github.com/k2-fsa/sherpa-onnx-go"
)

func main() {
	// 配置模型
	modelConfig := sherpaOnnx.OnlineModelConfig{
		Tokens: "./tokens.txt",
		Transducer: &sherpaOnnx.OnlineTransducerModelConfig{
			Encoder: "./encoder.onnx",
			Decoder: "./decoder.onnx",
			Joiner:  "./joiner.onnx",
		},
		NumThreads: 1,
		ModelType:  "zipformer",
	}

	// 创建识别器
	recognizerConfig := sherpaOnnx.OnlineRecognizerConfig{
		ModelConfig:      modelConfig,
		EnableEndpoint:   true,
		Rule1MinSilenceDuration: 0.5,
	}

	recognizer := sherpaOnnx.NewOnlineRecognizer(recognizerConfig)
	defer recognizer.Release()

	// 创建麦克风捕获器
	mic := sherpaOnnx.NewMicrophone(16000)
	defer mic.Release()

	// 处理中断信号
	sigChan := make(chan os.Signal, 1)
	signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)

	fmt.Println("开始录音，按 Ctrl+C 停止...")

	for {
		select {
		case <-sigChan:
			return
		default:
			// 读取音频
			samples := mic.Read(1600) // 100ms @ 16kHz
			if len(samples) == 0 {
				continue
			}

			// 创建流并识别
			stream := recognizer.CreateStream()
			defer stream.Release()

			stream.AcceptWaveform(samples)
			recognizer.Decode(stream)

			// 获取结果
			result := recognizer.Result(stream)
			if len(result.Text) > 0 {
				fmt.Printf("\r识别: %s", result.Text)
			}
		}
	}
}
```

### 语音合成

```go
// non-streaming-tts/main.go
package main

import (
	"fmt"
	"os"

	"github.com/k2-fsa/sherpa-onnx-go"
)

func main() {
	// 配置 TTS 模型
	modelConfig := sherpaOnnx.OfflineTtsModelConfig{
		Vits: &sherpaOnnx.OfflineTtsVitsModelConfig{
			Model:   "./vits.onnx",
			Lexicon: "./lexicon.txt",
			Tokens:  "./tokens.txt",
		},
		NumThreads: 1,
	}

	// 创建模型
	model := sherpaOnnx.NewOfflineTtsModel(modelConfig)
	defer model.Release()

	// 生成语音
	text := "Hello, world! 你好，世界！"
	audio := model.Generate(text, 0) // 说话人 ID = 0

	// 保存音频
	outputFile := "output.wav"
	err := audio.Save(outputFile)
	if err != nil {
		fmt.Printf("保存失败: %v\n", err)
		return
	}

	fmt.Printf("语音已保存到: %s\n", outputFile)
	fmt.Printf("采样率: %d Hz\n", audio.SampleRate)
	fmt.Printf("时长: %.2f 秒\n", float32(audio.NumFrames)/float32(audio.SampleRate))
}
```

### 说话人识别

```go
// speaker-identification/main.go
package main

import (
	"fmt"
	"math"

	"github.com/k2-fsa/sherpa-onnx-go"
)

func cosineSimilarity(x, y []float32) float32 {
	var dot, normX, normY float32
	for i := 0; i < len(x); i++ {
		dot += x[i] * y[i]
		normX += x[i] * x[i]
		normY += y[i] * y[i]
	}
	return dot / (float32(math.Sqrt(float64(normX))) * float32(math.Sqrt(float64(normY))))
}

func main() {
	// 配置模型
	modelConfig := sherpaOnnx.SpeakerEmbeddingExtractorModelConfig{
		Model:      "./speaker_model.onnx",
		NumThreads: 1,
	}

	// 创建提取器
	extractor := sherpaOnnx.NewSpeakerEmbeddingExtractor(modelConfig)
	defer extractor.Release()

	// 提取两个音频的说话人特征
	audio1 := "./speaker1.wav"
	audio2 := "./speaker2.wav"

	embedding1 := extractor.Compute(audio1)
	embedding2 := extractor.Compute(audio2)

	// 计算相似度
	similarity := cosineSimilarity(embedding1, embedding2)

	fmt.Printf("说话人相似度: %.2f\n", similarity)
	if similarity > 0.5 {
		fmt.Println("判断为同一说话人")
	} else {
		fmt.Println("判断为不同说话人")
	}
}
```

### 语音活动检测

```go
// vad/main.go
package main

import (
	"fmt"
	"log"

	"github.com/k2-fsa/sherpa-onnx-go"
)

func main() {
	// 配置 VAD 模型
	modelConfig := sherpaOnnx.VadModelConfig{
		SileroVad: &sherpaOnnx.SileroVadModelConfig{
			Model: "./silero_vad.onnx",
		},
		SampleRate: 16000,
		WindowSize: 512,
	}

	// 创建 VAD
	vad := sherpaOnnx.NewVad(modelConfig)
	defer vad.Release()

	// 读取音频
	audioFile := "./test.wav"
	waveData := sherpaOnnx.ReadWave(audioFile)

	// 检测语音活动
	stream := vad.CreateStream()
	defer stream.Release()

	stream.AcceptWaveform(waveData.Samples)
	vad.Compute(stream)

	// 获取结果
	segments := stream.Result()
	for i, segment := range segments {
		fmt.Printf("语音段 %d: %.2f - %.2f 秒\n",
			i, segment.Start, segment.End)
	}
}
```

## 构建和运行

### 1. 下载模型
```bash
# ASR 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2

# TTS 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-zh-hf-pan-shuang-ep.tar.bz2

# VAD 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/vad-models/silero_vad.onnx
```

### 2. 运行示例
```bash
cd non-streaming-decode-files
go run main.go

# 或构建后运行
go build -o asr
./asr
```

### 3. 跨平台编译
```bash
# Windows
GOOS=windows GOARCH=amd64 go build -o asr.exe main.go

# Linux
GOOS=linux GOARCH=amd64 go build -o asr main.go

# macOS
GOOS=darwin GOARCH=amd64 go build -o asr main.go
```

## 性能优化

### 1. 并发处理
利用 Go 的 goroutine 并发处理多个音频流：
```go
func processAudio(audioFile string, recognizer *sherpaOnnx.OfflineRecognizer) {
	// 并发处理
	go func() {
		stream := recognizer.NewStream()
		defer stream.Release()
		// ... 处理逻辑 ...
	}()
}
```

### 2. 缓冲区优化
调整音频缓冲区大小：
```go
// 根据延迟需求调整
bufferSize := 512   // 低延迟
bufferSize := 1600  // 平衡
bufferSize := 3200  // 高吞吐
```

### 3. 内存池
使用 sync.Pool 重用对象：
```go
var streamPool = sync.Pool{
	New: func() interface{} {
		return recognizer.CreateStream()
	},
}

func getStream() *sherpaOnnx.OfflineStream {
	return streamPool.Get().(*sherpaOnnx.OfflineStream)
}
```

## 常见问题

### 1. CGO 编译错误
确保安装了 C 编译器：
```bash
# macOS
xcode-select --install

# Linux
sudo apt-get install build-essential
```

### 2. 找不到 ONNX 库
设置库路径：
```bash
export LD_LIBRARY_PATH=/path/to/lib:$LD_LIBRARY_PATH
export DYLD_LIBRARY_PATH=/path/to/lib:$DYLD_LIBRARY_PATH
```

### 3. 音频设备问题
检查麦克风权限和设备：
```bash
# Linux
arecord -l

# macOS
sudo kextunload -i com.apple.driver.CoreAudio
sudo kextload -b com.apple.driver.CoreAudio
```

## 相关文档

- [Go API 文档](https://k2-fsa.github.io/sherpa/onnx/go-api/)
- [Sherpa-ONNX Go 模块](https://github.com/k2-fsa/sherpa-onnx-go)
- [Go 语言官网](https://golang.org/)

---

*更多示例和详细信息请参考：https://k2-fsa.github.io/sherpa/onnx/go-api/index.html*