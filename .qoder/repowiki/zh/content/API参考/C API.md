# C API

<cite>
**本文档引用的文件**   
- [c-api.h](file://sherpa-onnx/c-api/c-api.h)
- [c-api.cc](file://sherpa-onnx/c-api/c-api.cc)
- [decode-file-c-api.c](file://c-api-examples/decode-file-c-api.c)
- [kws-c-api.c](file://c-api-examples/kws-c-api.c)
- [offline-tts-c-api.c](file://c-api-examples/offline-tts-c-api.c)
- [offline-speaker-diarization-c-api.c](file://c-api-examples/offline-speaker-diarization-c-api.c)
- [audio-tagging-c-api.c](file://c-api-examples/audio-tagging-c-api.c)
</cite>

## 目录
1. [简介](#简介)
2. [结构体定义](#结构体定义)
3. [枚举类型和常量](#枚举类型和常量)
4. [语音识别](#语音识别)
5. [关键词识别](#关键词识别)
6. [语音合成](#语音合成)
7. [说话人识别](#说话人识别)
8. [语音活动检测](#语音活动检测)
9. [语音增强](#语音增强)
10. [音频标签](#音频标签)
11. [语言识别](#语言识别)
12. [标点符号添加](#标点符号添加)
13. [内存管理](#内存管理)
14. [线程安全性](#线程安全性)
15. [性能优化](#性能优化)
16. [高级语言绑定](#高级语言绑定)

## 简介

sherpa-onnx C API 提供了一个稳定、跨平台的接口，用于访问语音处理功能，包括语音识别、语音合成、说话人识别、关键词识别、语音活动检测等。该API设计为C语言风格，确保与各种编程语言的兼容性。

C API 的核心设计原则包括：
- **简单性**：提供直观的函数接口，易于集成
- **稳定性**：保持向后兼容的ABI（应用程序二进制接口）
- **内存安全**：明确的内存管理规则，避免内存泄漏
- **线程安全**：在多线程环境下的安全使用

API 的版本信息可以通过以下函数获取：

```c
// 获取版本字符串，例如 "1.12.1"
const char *SherpaOnnxGetVersionStr();

// 获取Git提交哈希
const char *SherpaOnnxGetGitSha1();

// 获取Git提交日期
const char *SherpaOnnxGetGitDate();
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L56-L73)

## 结构体定义

### 语音识别配置

#### 在线语音识别配置
```c
typedef struct SherpaOnnxOnlineRecognizerConfig {
  SherpaOnnxFeatureConfig feat_config;
  SherpaOnnxOnlineModelConfig model_config;
  const char *decoding_method;
  int32_t max_active_paths;
  int32_t enable_endpoint;
  float rule1_min_trailing_silence;
  float rule2_min_trailing_silence;
  float rule3_min_utterance_length;
  const char *hotwords_file;
  float hotwords_score;
  SherpaOnnxOnlineCtcFstDecoderConfig ctc_fst_decoder_config;
  const char *rule_fsts;
  const char *rule_fars;
  float blank_penalty;
  const char *hotwords_buf;
  int32_t hotwords_buf_size;
  SherpaOnnxHomophoneReplacerConfig hr;
} SherpaOnnxOnlineRecognizerConfig;
```

#### 离线语音识别配置
```c
typedef struct SherpaOnnxOfflineRecognizerConfig {
  SherpaOnnxFeatureConfig feat_config;
  SherpaOnnxOfflineModelConfig model_config;
  SherpaOnnxOfflineLMConfig lm_config;
  const char *decoding_method;
  int32_t max_active_paths;
  const char *hotwords_file;
  float hotwords_score;
  const char *rule_fsts;
  const char *rule_fars;
  float blank_penalty;
  SherpaOnnxHomophoneReplacerConfig hr;
} SherpaOnnxOfflineRecognizerConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L158-L204)
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L516-L534)

### 模型配置

#### 在线模型配置
```c
typedef struct SherpaOnnxOnlineModelConfig {
  SherpaOnnxOnlineTransducerModelConfig transducer;
  SherpaOnnxOnlineParaformerModelConfig paraformer;
  SherpaOnnxOnlineZipformer2CtcModelConfig zipformer2_ctc;
  const char *tokens;
  int32_t num_threads;
  const char *provider;
  int32_t debug;
  const char *model_type;
  const char *modeling_unit;
  const char *bpe_vocab;
  const char *tokens_buf;
  int32_t tokens_buf_size;
  SherpaOnnxOnlineNemoCtcModelConfig nemo_ctc;
  SherpaOnnxOnlineToneCtcModelConfig t_one_ctc;
} SherpaOnnxOnlineModelConfig;
```

#### 离线模型配置
```c
typedef struct SherpaOnnxOfflineModelConfig {
  SherpaOnnxOfflineTransducerModelConfig transducer;
  SherpaOnnxOfflineParaformerModelConfig paraformer;
  SherpaOnnxOfflineNemoEncDecCtcModelConfig nemo_ctc;
  SherpaOnnxOfflineWhisperModelConfig whisper;
  SherpaOnnxOfflineTdnnModelConfig tdnn;
  const char *tokens;
  int32_t num_threads;
  int32_t debug;
  const char *provider;
  const char *model_type;
  const char *modeling_unit;
  const char *bpe_vocab;
  const char *telespeech_ctc;
  SherpaOnnxOfflineSenseVoiceModelConfig sense_voice;
  SherpaOnnxOfflineMoonshineModelConfig moonshine;
  SherpaOnnxOfflineFireRedAsrModelConfig fire_red_asr;
  SherpaOnnxOfflineDolphinModelConfig dolphin;
  SherpaOnnxOfflineZipformerCtcModelConfig zipformer_ctc;
  SherpaOnnxOfflineCanaryModelConfig canary;
  SherpaOnnxOfflineWenetCtcModelConfig wenet_ctc;
  SherpaOnnxOfflineOmnilingualAsrCtcModelConfig omnilingual;
} SherpaOnnxOfflineModelConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L111-L133)
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L487-L514)

### 特征配置
```c
typedef struct SherpaOnnxFeatureConfig {
  int32_t sample_rate;
  int32_t feature_dim;
} SherpaOnnxFeatureConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L135-L145)

### 语音合成配置
```c
typedef struct SherpaOnnxOfflineTtsConfig {
  SherpaOnnxOfflineTtsModelConfig model;
  const char *rule_fsts;
  int32_t max_num_sentences;
  const char *rule_fars;
  float silence_scale;
} SherpaOnnxOfflineTtsConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1093-L1099)

### 说话人识别配置
```c
typedef struct SherpaOnnxSpeakerEmbeddingExtractorConfig {
  const char *model;
  int32_t num_threads;
  int32_t debug;
  const char *provider;
} SherpaOnnxSpeakerEmbeddingExtractorConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1290-L1295)

### 语音活动检测配置
```c
typedef struct SherpaOnnxVadModelConfig {
  SherpaOnnxSileroVadModelConfig silero_vad;
  int32_t sample_rate;
  int32_t num_threads;
  const char *provider;
  int32_t debug;
  SherpaOnnxTenVadModelConfig ten_vad;
} SherpaOnnxVadModelConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L905-L913)

### 音频标签配置
```c
typedef struct SherpaOnnxAudioTaggingConfig {
  SherpaOnnxAudioTaggingModelConfig model;
  const char *labels;
  int32_t top_k;
} SherpaOnnxAudioTaggingConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1499-L1503)

### 说话人分离配置
```c
typedef struct SherpaOnnxOfflineSpeakerDiarizationConfig {
  SherpaOnnxOfflineSpeakerSegmentationModelConfig segmentation;
  SherpaOnnxSpeakerEmbeddingExtractorConfig embedding;
  SherpaOnnxFastClusteringConfig clustering;
  float min_duration_on;
  float min_duration_off;
} SherpaOnnxOfflineSpeakerDiarizationConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1689-L1701)

### 语音增强配置
```c
typedef struct SherpaOnnxOfflineSpeechDenoiserConfig {
  SherpaOnnxOfflineSpeechDenoiserModelConfig model;
} SherpaOnnxOfflineSpeechDenoiserConfig;
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1797-L1799)

## 枚举类型和常量

### 解码方法
支持的解码方法包括：
- `greedy_search`：贪心搜索
- `modified_beam_search`：改进的束搜索

### 提供者（Provider）
支持的计算后端：
- `cpu`：CPU计算
- `cuda`：NVIDIA GPU计算
- `coreml`：Apple Core ML框架

### 建模单元
支持的建模单元：
- `cjkchar`：中日韩字符
- `bpe`：字节对编码
- `cjkchar+bpe`：中日韩字符与BPE结合

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L120-L124)
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L499-L503)

## 语音识别

### 在线语音识别

在线语音识别（流式识别）允许实时处理音频流。

#### 创建识别器
```c
const SherpaOnnxOnlineRecognizer *SherpaOnnxCreateOnlineRecognizer(
    const SherpaOnnxOnlineRecognizerConfig *config);
```

#### 创建流
```c
const SherpaOnnxOnlineStream *SherpaOnnxCreateOnlineStream(
    const SherpaOnnxOnlineRecognizer *recognizer);
```

#### 接受音频数据
```c
void SherpaOnnxOnlineStreamAcceptWaveform(
    const SherpaOnnxOnlineStream *stream,
    int32_t sample_rate,
    const float *samples,
    int32_t n);
```

#### 解码
```c
int32_t SherpaOnnxIsOnlineStreamReady(
    const SherpaOnnxOnlineRecognizer *recognizer,
    const SherpaOnnxOnlineStream *stream);

void SherpaOnnxDecodeOnlineStream(
    const SherpaOnnxOnlineRecognizer *recognizer,
    const SherpaOnnxOnlineStream *stream);
```

#### 获取结果
```c
const SherpaOnnxOnlineRecognizerResult *SherpaOnnxGetOnlineStreamResult(
    const SherpaOnnxOnlineRecognizer *recognizer,
    const SherpaOnnxOnlineStream *stream);
```

#### 销毁结果
```c
void SherpaOnnxDestroyOnlineRecognizerResult(
    const SherpaOnnxOnlineRecognizerResult *r);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L249-L354)

### 离线语音识别

离线语音识别（非流式识别）用于处理完整的音频文件。

#### 创建识别器
```c
const SherpaOnnxOfflineRecognizer *SherpaOnnxCreateOfflineRecognizer(
    const SherpaOnnxOfflineRecognizerConfig *config);
```

#### 创建流
```c
const SherpaOnnxOfflineStream *SherpaOnnxCreateOfflineStream(
    const SherpaOnnxOfflineRecognizer *recognizer);
```

#### 接受音频数据
```c
void SherpaOnnxAcceptWaveformOffline(
    const SherpaOnnxOfflineStream *stream,
    int32_t sample_rate,
    const float *samples,
    int32_t n);
```

#### 解码
```c
void SherpaOnnxDecodeOfflineStream(
    const SherpaOnnxOfflineRecognizer *recognizer,
    const SherpaOnnxOfflineStream *stream);
```

#### 获取结果
```c
const SherpaOnnxOfflineRecognizerResult *SherpaOnnxGetOfflineStreamResult(
    const SherpaOnnxOfflineStream *stream);
```

#### 销毁结果
```c
void SherpaOnnxDestroyOfflineRecognizerResult(
    const SherpaOnnxOfflineRecognizerResult *r);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L541-L693)

### 调用示例

```c
// 读取音频文件
const SherpaOnnxWave *wave = SherpaOnnxReadWave(wav_filename);

// 创建识别器和流
const SherpaOnnxOnlineRecognizer *recognizer = 
    SherpaOnnxCreateOnlineRecognizer(&config);
const SherpaOnnxOnlineStream *stream = 
    SherpaOnnxCreateOnlineStream(recognizer);

// 模拟流式处理
while (k < wave->num_samples) {
    // 接受音频数据
    SherpaOnnxOnlineStreamAcceptWaveform(stream, wave->sample_rate,
                                       wave->samples + start, end - start);
    
    // 解码
    while (SherpaOnnxIsOnlineStreamReady(recognizer, stream)) {
        SherpaOnnxDecodeOnlineStream(recognizer, stream);
    }
    
    // 获取结果
    const SherpaOnnxOnlineRecognizerResult *r = 
        SherpaOnnxGetOnlineStreamResult(recognizer, stream);
    
    if (strlen(r->text)) {
        printf("识别结果: %s\n", r->text);
    }
    
    SherpaOnnxDestroyOnlineRecognizerResult(r);
}

// 清理资源
SherpaOnnxFreeWave(wave);
SherpaOnnxDestroyOnlineStream(stream);
SherpaOnnxDestroyOnlineRecognizer(recognizer);
```

**Section sources**
- [decode-file-c-api.c](file://c-api-examples/decode-file-c-api.c#L165-L240)

## 关键词识别

关键词识别用于检测音频流中的特定关键词。

### 创建关键词识别器
```c
const SherpaOnnxKeywordSpotter *SherpaOnnxCreateKeywordSpotter(
    const SherpaOnnxKeywordSpotterConfig *config);
```

### 创建流
```c
const SherpaOnnxOnlineStream *SherpaOnnxCreateKeywordStream(
    const SherpaOnnxKeywordSpotter *spotter);
```

### 解码
```c
void SherpaOnnxDecodeKeywordStream(
    const SherpaOnnxKeywordSpotter *spotter,
    const SherpaOnnxOnlineStream *stream);
```

### 获取结果
```c
const SherpaOnnxKeywordResult *SherpaOnnxGetKeywordResult(
    const SherpaOnnxKeywordSpotter *spotter,
    const SherpaOnnxOnlineStream *stream);
```

### 重置流
```c
void SherpaOnnxResetKeywordStream(
    const SherpaOnnxKeywordSpotter *spotter,
    const SherpaOnnxOnlineStream *stream);
```

### 调用示例
```c
// 创建关键词识别器
const SherpaOnnxKeywordSpotter *kws = SherpaOnnxCreateKeywordSpotter(&config);

// 创建流
const SherpaOnnxOnlineStream *stream = SherpaOnnxCreateKeywordStream(kws);

// 处理音频
SherpaOnnxOnlineStreamAcceptWaveform(stream, wave->sample_rate, 
                                   wave->samples, wave->num_samples);

// 解码
while (SherpaOnnxIsKeywordStreamReady(kws, stream)) {
    SherpaOnnxDecodeKeywordStream(kws, stream);
    const SherpaOnnxKeywordResult *r = SherpaOnnxGetKeywordResult(kws, stream);
    
    if (r && strlen(r->keyword)) {
        printf("检测到关键词: %s\n", r->keyword);
        SherpaOnnxResetKeywordStream(kws, stream);
    }
    
    SherpaOnnxDestroyKeywordResult(r);
}

// 清理资源
SherpaOnnxDestroyOnlineStream(stream);
SherpaOnnxDestroyKeywordSpotter(kws);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L764-L843)
- [kws-c-api.c](file://c-api-examples/kws-c-api.c#L51-L147)

## 语音合成

语音合成将文本转换为语音。

### 创建TTS引擎
```c
const SherpaOnnxOfflineTts *SherpaOnnxCreateOfflineTts(
    const SherpaOnnxOfflineTtsConfig *config);
```

### 生成语音
```c
const SherpaOnnxGeneratedAudio *SherpaOnnxOfflineTtsGenerate(
    const SherpaOnnxOfflineTts *tts,
    const char *text,
    int32_t sid,
    float speed);
```

### 销毁生成的音频
```c
void SherpaOnnxDestroyOfflineTtsGeneratedAudio(
    const SherpaOnnxGeneratedAudio *p);
```

### 写入波形文件
```c
int32_t SherpaOnnxWriteWave(
    const float *samples,
    int32_t n,
    int32_t sample_rate,
    const char *filename);
```

### 调用示例
```c
// 创建TTS引擎
const SherpaOnnxOfflineTts *tts = SherpaOnnxCreateOfflineTts(&config);

// 生成语音
const SherpaOnnxGeneratedAudio *audio = SherpaOnnxOfflineTtsGenerate(tts, 
                                                                   "你好世界", 
                                                                   0, 
                                                                   1.0f);

// 写入文件
SherpaOnnxWriteWave(audio->samples, audio->n, audio->sample_rate, "output.wav");

// 清理资源
SherpaOnnxDestroyOfflineTtsGeneratedAudio(audio);
SherpaOnnxDestroyOfflineTts(tts);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1126-L1185)
- [offline-tts-c-api.c](file://c-api-examples/offline-tts-c-api.c)

## 说话人识别

说话人识别用于识别说话人的身份。

### 创建提取器
```c
const SherpaOnnxSpeakerEmbeddingExtractor *SherpaOnnxCreateSpeakerEmbeddingExtractor(
    const SherpaOnnxSpeakerEmbeddingExtractorConfig *config);
```

### 计算嵌入
```c
const float *SherpaOnnxSpeakerEmbeddingExtractorComputeEmbedding(
    const SherpaOnnxSpeakerEmbeddingExtractor *p,
    const SherpaOnnxOnlineStream *s);
```

### 创建管理器
```c
const SherpaOnnxSpeakerEmbeddingManager *SherpaOnnxCreateSpeakerEmbeddingManager(
    int32_t dim);
```

### 添加说话人
```c
int32_t SherpaOnnxSpeakerEmbeddingManagerAdd(
    const SherpaOnnxSpeakerEmbeddingManager *p,
    const char *name,
    const float *v);
```

### 验证说话人
```c
int32_t SherpaOnnxSpeakerEmbeddingManagerVerify(
    const SherpaOnnxSpeakerEmbeddingManager *p,
    const char *name,
    const float *v,
    float threshold);
```

### 调用示例
```c
// 创建提取器
const SherpaOnnxSpeakerEmbeddingExtractor *extractor = 
    SherpaOnnxCreateSpeakerEmbeddingExtractor(&config);

// 创建管理器
const SherpaOnnxSpeakerEmbeddingManager *manager = 
    SherpaOnnxCreateSpeakerEmbeddingManager(dim);

// 提取参考说话人嵌入
const float *embedding = 
    SherpaOnnxSpeakerEmbeddingExtractorComputeEmbedding(extractor, stream);

// 注册说话人
SherpaOnnxSpeakerEmbeddingManagerAdd(manager, "speaker1", embedding);

// 验证新说话人
int32_t is_match = SherpaOnnxSpeakerEmbeddingManagerVerify(manager, 
                                                           "speaker1", 
                                                           new_embedding, 
                                                           0.6f);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1303-L1451)
- [offline-speaker-diarization-c-api.c](file://c-api-examples/offline-speaker-diarization-c-api.c)

## 语音活动检测

语音活动检测（VAD）用于检测音频中的语音段。

### 创建VAD检测器
```c
const SherpaOnnxVoiceActivityDetector *SherpaOnnxCreateVoiceActivityDetector(
    const SherpaOnnxVadModelConfig *config,
    float buffer_size_in_seconds);
```

### 接受音频数据
```c
void SherpaOnnxVoiceActivityDetectorAcceptWaveform(
    const SherpaOnnxVoiceActivityDetector *p,
    const float *samples,
    int32_t n);
```

### 获取语音段
```c
const SherpaOnnxSpeechSegment *SherpaOnnxVoiceActivityDetectorFront(
    const SherpaOnnxVoiceActivityDetector *p);
```

### 弹出语音段
```c
void SherpaOnnxVoiceActivityDetectorPop(
    const SherpaOnnxVoiceActivityDetector *p);
```

### 调用示例
```c
// 创建VAD检测器
const SherpaOnnxVoiceActivityDetector *vad = 
    SherpaOnnxCreateVoiceActivityDetector(&config, 10.0f);

// 处理音频
SherpaOnnxVoiceActivityDetectorAcceptWaveform(vad, samples, n);

// 获取语音段
while (!SherpaOnnxVoiceActivityDetectorEmpty(vad)) {
    const SherpaOnnxSpeechSegment *segment = 
        SherpaOnnxVoiceActivityDetectorFront(vad);
    
    // 处理语音段
    process_segment(segment->samples, segment->n);
    
    // 弹出已处理的段
    SherpaOnnxVoiceActivityDetectorPop(vad);
    SherpaOnnxDestroySpeechSegment(segment);
}
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L975-L1012)

## 语音增强

语音增强用于去除音频中的噪声。

### 创建语音增强器
```c
const SherpaOnnxOfflineSpeechDenoiser *SherpaOnnxCreateOfflineSpeechDenoiser(
    const SherpaOnnxOfflineSpeechDenoiserConfig *config);
```

### 运行语音增强
```c
const SherpaOnnxDenoisedAudio *SherpaOnnxOfflineSpeechDenoiserRun(
    const SherpaOnnxOfflineSpeechDenoiser *sd,
    const float *samples,
    int32_t n,
    int32_t sample_rate);
```

### 销毁去噪音频
```c
void SherpaOnnxDestroyDenoisedAudio(
    const SherpaOnnxDenoisedAudio *p);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1807-L1837)

## 音频标签

音频标签用于识别音频中的声音事件。

### 创建音频标签器
```c
const SherpaOnnxAudioTagging *SherpaOnnxCreateAudioTagging(
    const SherpaOnnxAudioTaggingConfig *config);
```

### 计算标签
```c
const SherpaOnnxAudioEvent *const *SherpaOnnxAudioTaggingCompute(
    const SherpaOnnxAudioTagging *tagger,
    const SherpaOnnxOfflineStream *s,
    int32_t top_k);
```

### 释放结果
```c
void SherpaOnnxAudioTaggingFreeResults(
    const SherpaOnnxAudioEvent *const *p);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1516-L1541)

## 语言识别

语言识别用于确定音频中的语言。

### 创建语言识别器
```c
const SherpaOnnxSpokenLanguageIdentification *SherpaOnnxCreateSpokenLanguageIdentification(
    const SherpaOnnxSpokenLanguageIdentificationConfig *config);
```

### 计算语言
```c
const SherpaOnnxSpokenLanguageIdentificationResult *SherpaOnnxSpokenLanguageIdentificationCompute(
    const SherpaOnnxSpokenLanguageIdentification *slid,
    const SherpaOnnxOfflineStream *s);
```

### 销毁结果
```c
void SherpaOnnxDestroySpokenLanguageIdentificationResult(
    const SherpaOnnxSpokenLanguageIdentificationResult *r);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1256-L1285)

## 标点符号添加

标点符号添加用于为文本添加适当的标点。

### 创建标点处理器
```c
const SherpaOnnxOfflinePunctuation *SherpaOnnxCreateOfflinePunctuation(
    const SherpaOnnxOfflinePunctuationConfig *config);
```

### 添加标点
```c
const char *SherpaOfflinePunctuationAddPunct(
    const SherpaOnnxOfflinePunctuation *punct,
    const char *text);
```

### 释放文本
```c
void SherpaOfflinePunctuationFreeText(
    const char *text);
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L1564-L1576)

## 内存管理

C API 遵循明确的内存管理规则，所有动态分配的内存都必须由调用者显式释放。

### 对象创建和销毁
- 所有 `Create` 函数返回的对象必须通过相应的 `Destroy` 函数释放
- 所有 `Get` 函数返回的结果必须通过相应的 `Destroy` 函数释放
- 从函数返回的字符串指针通常需要通过相应的 `Free` 函数释放

### 内存管理函数
```c
// 识别器
void SherpaOnnxDestroyOnlineRecognizer(const SherpaOnnxOnlineRecognizer *recognizer);
void SherpaOnnxDestroyOfflineRecognizer(const SherpaOnnxOfflineRecognizer *recognizer);

// 流
void SherpaOnnxDestroyOnlineStream(const SherpaOnnxOnlineStream *stream);
void SherpaOnnxDestroyOfflineStream(const SherpaOnnxOfflineStream *stream);

// 结果
void SherpaOnnxDestroyOnlineRecognizerResult(const SherpaOnnxOnlineRecognizerResult *r);
void SherpaOnnxDestroyOfflineRecognizerResult(const SherpaOnnxOfflineRecognizerResult *r);

// 音频
void SherpaOnnxFreeWave(const SherpaOnnxWave *wave);
```

### 内存管理最佳实践
1. 总是成对使用创建和销毁函数
2. 在函数返回前确保释放所有临时分配的内存
3. 使用RAII模式或智能指针（在C++中）来管理资源
4. 避免内存泄漏和双重释放

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L259-L260)
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L557-L558)

## 线程安全性

C API 的线程安全性遵循以下原则：

### 线程安全函数
- 所有创建函数（`Create*`）是线程安全的
- 所有销毁函数（`Destroy*`）是线程安全的
- 不同对象实例之间的函数调用是线程安全的

### 非线程安全情况
- 同一对象实例的多个线程同时调用非const方法是非线程安全的
- 同一识别器实例的多个流同时解码需要同步

### 线程安全使用模式
```c
// 线程安全：不同线程使用不同的识别器实例
void thread_function() {
    const SherpaOnnxOnlineRecognizer *recognizer = 
        SherpaOnnxCreateOnlineRecognizer(&config);
    
    // 使用识别器...
    
    SherpaOnnxDestroyOnlineRecognizer(recognizer);
}

// 非线程安全：多个线程共享同一识别器实例
// 需要使用互斥锁保护
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void unsafe_thread_function() {
    pthread_mutex_lock(&mutex);
    // 使用共享的识别器实例
    pthread_mutex_unlock(&mutex);
}
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h)

## 性能优化

### 多线程处理
通过设置 `num_threads` 参数来利用多核CPU：

```c
config.model_config.num_threads = 4; // 使用4个线程
```

### 批量处理
使用批量解码函数提高吞吐量：

```c
// 批量解码多个流
void SherpaOnnxDecodeMultipleOnlineStreams(
    const SherpaOnnxOnlineRecognizer *recognizer,
    const SherpaOnnxOnlineStream **streams,
    int32_t n);
```

### GPU加速
使用CUDA提供者进行GPU加速：

```c
config.model_config.provider = "cuda"; // 使用CUDA
```

### 内存池
对于频繁创建和销毁的对象，考虑使用对象池来减少内存分配开销。

### 预热
在生产环境中，建议在处理实际请求前进行预热：

```c
// 预热模型
void warmup_model() {
    const SherpaOnnxOnlineRecognizer *recognizer = 
        SherpaOnnxCreateOnlineRecognizer(&config);
    const SherpaOnnxOnlineStream *stream = 
        SherpaOnnxCreateOnlineStream(recognizer);
    
    // 处理一些测试数据
    float test_samples[1600]; // 0.1秒音频
    memset(test_samples, 0, sizeof(test_samples));
    SherpaOnnxOnlineStreamAcceptWaveform(stream, 16000, test_samples, 1600);
    
    while (SherpaOnnxIsOnlineStreamReady(recognizer, stream)) {
        SherpaOnnxDecodeOnlineStream(recognizer, stream);
    }
    
    SherpaOnnxDestroyOnlineStream(stream);
    SherpaOnnxDestroyOnlineRecognizer(recognizer);
}
```

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L116-L117)
- [c-api.h](file://sherpa-onnx/c-api/c-api.h#L496-L497)

## 高级语言绑定

C API 是所有高级语言绑定的基础，包括：

### Python绑定
```python
import sherpa_onnx

# 使用C API的Python封装
recognizer = sherpa_onnx.OnlineRecognizer(
    config=sherpa_onnx.OnlineRecognizerConfig(
        model_config=sherpa_onnx.OnlineModelConfig(
            transducer=sherpa_onnx.OnlineTransducerModelConfig(
                encoder="encoder.onnx",
                decoder="decoder.onnx",
                joiner="joiner.onnx",
            ),
            tokens="tokens.txt",
        ),
    )
)
```

### Java绑定
```java
// Java通过JNI调用C API
SherpaOnnxOnlineRecognizer recognizer = new SherpaOnnxOnlineRecognizer(config);
```

### .NET绑定
```csharp
// .NET通过P/Invoke调用C API
[DllImport("sherpa-onnx-c-api")]
public static extern IntPtr SherpaOnnxCreateOnlineRecognizer(
    ref SherpaOnnxOnlineRecognizerConfig config);
```

### Node.js绑定
```javascript
// Node.js通过node-addon-api调用C API
const { createOnlineRecognizer } = require('sherpa-onnx');
```

所有高级语言绑定都通过C API与底层实现交互，确保功能的一致性和稳定性。

**Section sources**
- [c-api.h](file://sherpa-onnx/c-api/c-api.h)
- [c-api.cc](file://sherpa-onnx/c-api/c-api.cc)