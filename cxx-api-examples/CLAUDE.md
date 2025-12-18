# C++ API 示例

> 更新时间：2025-12-18 10:27:00

## 模块概述

本模块包含 Sherpa-ONNX C++ API 的使用示例，展示了如何使用 C++ 接口实现各种语音处理功能。这些示例是学习和理解 Sherpa-ONNX C++ API 的最佳起点。

## 功能分类

### 🎤 语音识别 (ASR) 示例

#### 流式识别
- **streaming-zipformer-cxx-api.cc**: 使用 Zipformer 模型进行实时流式语音识别
- **streaming-zipformer-with-hr-cxx-api.cc**: 带热词增强的流式识别
- **streaming-zipformer-rtf-cxx-api.cc**: 实时因子（RTF）性能测试示例
- **streaming-t-one-ctc-cxx-api.cc**: 使用 TeleSpeech T-One CTC 模型
- **sense-voice-simulate-streaming-microphone-cxx-api.cc**: SenseVoice 模拟麦克风输入
- **sense-voice-simulate-streaming-alsa-cxx-api.cc**: SenseVoice ALSA 音频输入
- **sense-voice-with-hr-cxx-api.cc**: SenseVoice 带热词支持
- **parakeet-tdt-simulate-streaming-microphone-cxx-api.cc**: Parakeet TDT 实时识别
- **parakeet-tdt-ctc-simulate-streaming-microphone-cxx-api.cc**: Parakeet TDT CTT 版本
- **zipformer-transducer-simulate-streaming-microphone-cxx-api.cc**: Zipformer Transducer 模型
- **zipformer-ctc-simulate-streaming-microphone-cxx-api.cc**: Zipformer CTC 模型
- **zipformer-ctc-simulate-streaming-alsa-cxx-api.cc**: Zipformer ALSA 音频输入
- **wenet-ctc-simulate-streaming-microphone-cxx-api.cc**: WeNet CTC 实时识别

#### 非流式识别
- **whisper-cxx-api.cc**: OpenAI Whisper 模型示例
- **sense-voice-cxx-api.cc**: SenseVoice 多语言识别
- **fire-red-asr-cxx-api.cc**: FireRed ASR 中文优化模型
- **moonshine-cxx-api.cc**: Moonshine 轻量级模型
- **wenet-ctc-cxx-api.cc**: WeNet CTC 批量识别
- **omnilingual-asr-ctc-cxx-api.cc**: Omnilingual 多语言 CTC
- **nemo-canary-cxx-api.cc**: NVIDIA NeMo Canary 模型
- **dolphin-ctc-cxx-api.cc**: Dolphin CTT 模型

### 🔊 语音合成 (TTS) 示例
- **matcha-tts-en-cxx-api.cc**: Matcha-TTS 英语合成
- **matcha-tts-zh-cxx-api.cc**: Matcha-TTS 中文合成
- **kokoro-tts-en-cxx-api.cc**: Kokoro 高质量英语 TTS
- **kokoro-tts-zh-en-cxx-api.cc**: Kokoro 中英双语 TTS
- **kitten-tts-en-cxx-api.cc**: Kitten TTS 英语合成

### 🎯 其他功能示例

#### 关键词检测 (KWS)
- **kws-cxx-api.cc**: 关键词唤醒词检测

#### 语音活动检测 (VAD)
- **vad-cxx-api.cc**: 语音活动检测

#### 语音增强
- **speech-enhancement-gtcrn-cxx-api.cc**: GTCRN 语音增强

#### 音频标注
- **audio-tagging-ced-cxx-api.cc**: CED 音频分类标注
- **audio-tagging-zipformer-cxx-api.cc**: Zipformer 音频标注

#### 标点恢复
- **offline-punctuation-cxx-api.cc**: 离线标点符号恢复
- **online-punctuation-cxx-api.cc**: 在线标点符号恢复

## 构建方法

所有示例都通过 CMake 构建：

```bash
# 在 sherpa-onnx 根目录
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j4 streaming-zipformer-cxx-api  # 构建特定示例
make -j4 cxx-api-examples             # 构建所有 C++ 示例
```

## 使用示例

### 基本流式识别

```cpp
#include "sherpa-onnx/c-api/cxx-api.h"

int main() {
    using namespace sherpa_onnx::cxx;

    // 配置识别器
    OnlineRecognizerConfig config;
    config.model_config.transducer.encoder = "encoder.onnx";
    config.model_config.transducer.decoder = "decoder.onnx";
    config.model_config.transducer.joiner = "joiner.onnx";
    config.model_config.tokens = "tokens.txt";

    // 创建识别器
    OnlineRecognizer recognizer = OnlineRecognizer::Create(config);

    // 创建流
    OnlineStream stream = recognizer.CreateStream();

    // 模拟音频输入
    std::vector<float> audio_samples = /* 音频数据 */;
    stream.AcceptWaveform(16000, audio_samples.data(), audio_samples.size());

    // 识别结果
    recognizer.Decode(stream);
    std::string result = recognizer.GetResult(stream);

    std::cout << "识别结果: " << result << std::endl;

    return 0;
}
```

### 关键词检测

```cpp
#include "sherpa-onnx/c-api/cxx-api.h"

int main() {
    using namespace sherpa_onnx::cxx;

    // 配置关键词检测器
    KeywordSpotterConfig config;
    config.model_config.transducer.encoder = "encoder.onnx";
    config.model_config.transducer.decoder = "decoder.onnx";
    config.model_config.transducer.joiner = "joiner.onnx";
    config.model_config.tokens = "tokens.txt";
    config.keywords_file = "keywords.txt";
    config.threshold = 0.5;

    // 创建检测器
    KeywordSpotter spotter = KeywordSpotter::Create(config);

    // 处理音频流
    OnlineStream stream = spotter.CreateStream();
    // ... 音频处理 ...

    // 检测关键词
    auto result = spotter.StreamDetect(stream);
    if (result.keyword != "") {
        std::cout << "检测到关键词: " << result.keyword << std::endl;
    }

    return 0;
}
```

## 模型下载

示例中使用的模型可以从 Sherpa-ONNX releases 下载：

```bash
# Zipformer 双语模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2

# SenseVoice 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17.tar.bz2

# Whisper 模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-whisper-base.tar.bz2
```

## 性能优化建议

1. **线程配置**: 通过 `config.model_config.num_threads` 设置线程数
2. **模型量化**: 使用 int8 量化模型减少内存占用
3. **批处理**: 对于批量处理，使用非流式 API
4. **音频缓冲**: 合理设置音频缓冲区大小

## 扩展开发

基于这些示例，您可以：
- 集成到现有的 C++ 应用中
- 开发自定义的语音处理流程
- 创建实时语音交互系统
- 构建语音命令控制应用

## 相关文档

- [Sherpa-ONNX C++ API 文档](https://k2-fsa.github.io/sherpa/onnx/cxx-api/)
- [预训练模型列表](https://k2-fsa.github.io/sherpa/onnx/pretrained_models/index.html)
- [C++ 核心模块](../sherpa-onnx/csrc/CLAUDE.md)

---

*更多示例和详细信息请参考 Sherpa-ONNX 官方文档：https://github.com/k2-fsa/sherpa-onnx*