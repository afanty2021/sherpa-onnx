[根目录](../CLAUDE.md) > **c-api-examples**

# C API 示例

> 更新时间：2025-12-10 07:44:45

## 模块职责

本模块包含使用 Sherpa-ONNX C API 的示例程序，展示了如何在 C/C++ 中调用各种语音处理功能。这些示例的特点：

1. **跨平台兼容**：支持 Linux、Windows、macOS
2. **零依赖**：仅依赖标准库和 Sherpa-ONNX
3. **高性能**：直接调用 C++ 核心，无额外开销
4. **易于集成**：可作为模板集成到其他项目

## 示例分类

### 🎤 语音识别 (ASR) 示例

#### 基础识别
- `decode-file-c-api.c` - 文件解码基础示例
- `run-decode-file-c-api.sh` - 编译和运行脚本

#### 流式识别
- `streaming-zipformer-c-api.c` - Zipformer 流式识别
- `streaming-paraformer-c-api.c` - Paraformer 流式识别
- `streaming-paraformer-buffered-tokens-c-api.c` - 带缓冲的流式识别

#### CTC 模型
- `streaming-ctc-buffered-tokens-c-api.c` - CTC 流式识别
- `streaming-zipformer-buffered-tokens-hotwords-c-api.c` - 支持热词
- `streaming-t-one-ctc-c-api.c` - T-One CTC 模型
- `streaming-hlg-decode-file-c-api.c` - HLG 解码

#### 非流式识别
- `paraformer-c-api.c` - Paraformer 非流式
- `whisper-c-api.c` - Whisper 模型
- `sense-voice-c-api.c` - SenseVoice 多语言
- `zipformer-c-api.c` - Zipformer 非流式

#### 特殊模型
- `moonshine-c-api.c` - Moonshine 轻量模型
- `nemo-parakeet-c-api.c` - NVIDIA NeMo
- `telespeech-c-api.c` - 科大讯飞 TeleSpeech
- `wenet-ctc-c-api.c` - WeNet CTC
- `omnilingual-asr-ctc-c-api.c` - 1600 语言模型
- `dolphin-ctc-c-api.c` - Dolphin 多语言

### 🔊 TTS 示例
- `offline-tts-c-api.c` - 基础 TTS
- `matcha-tts-zh-c-api.c` - Matcha 中文 TTS
- `matcha-tts-en-c-api.c` - Matcha 英文 TTS
- `kokoro-tts-en-c-api.c` - Kokoro 英文 TTS
- `kokoro-tts-zh-en-c-api.c` - Kokoro 中英混合
- `kitten-tts-en-c-api.c` - Kitten 轻量 TTS

### 👥 说话人处理
- `speaker-identification-c-api.c` - 说话人识别
- `offline-speaker-diarization-c-api.c` - 说话人日志

### 🎯 其他功能
- `kws-c-api.c` - 关键词检测
- `audio-tagging-c-api.c` - 音频标记
- `speech-enhancement-gtcrn-c-api.c` - GT-CRN 语音增强
- `spoken-language-identification-c-api.c` - 语言识别
- `add-punctuation-c-api.c` - 标点恢复

### 🔀 VAD + ASR 组合
- `vad-moonshine-c-api.c` - VAD + Moonshine
- `vad-sense-voice-c-api.c` - VAD + SenseVoice
- `vad-whisper-c-api.c` - VAD + Whisper
- `sense-voice-with-hr-c-api.c` - SenseVoice + 语音恢复

## 基础示例解析

### decode-file-c-api.c 核心代码
```c
#include "sherpa-onnx/c-api.h"
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "Usage: %s model_dir wav_file\n", argv[0]);
        return -1;
    }

    // 1. 创建配置
    SherpaOnnxOfflineRecognizerConfig config;
    memset(&config, 0, sizeof(config));
    config.model_config.transducer.encoder = argv[1];
    config.model_config.transducer.decoder = argv[1];
    config.model_config.transducer.joiner = argv[1];
    config.model_config.tokens = argv[1];
    config.num_threads = 2;
    config.decoding_method = "greedy_search";

    // 2. 创建识别器
    SherpaOnnxRecognizer *recognizer = SherpaOnnxCreateOfflineRecognizer(&config);
    if (!recognizer) {
        fprintf(stderr, "Failed to create recognizer\n");
        return -1;
    }

    // 3. 读取音频
    SherpaOnnxWave *wave = SherpaOnnxReadWave(argv[2]);
    if (!wave) {
        fprintf(stderr, "Failed to read wave file: %s\n", argv[2]);
        SherpaOnnxDestroyOfflineRecognizer(recognizer);
        return -1;
    }

    // 4. 创建流
    SherpaOnnxOfflineStream *stream = SherpaOnnxCreateOfflineStream(recognizer);
    SherpaOnnxAcceptWaveformOffline(stream, wave->sample_rate,
                                   wave->samples, wave->num_samples);

    // 5. 识别
    SherpaOnnxDecodeOfflineStream(recognizer, stream);

    // 6. 获取结果
    SherpaOnnxResult *result = SherpaOnnxGetOfflineStreamResult(stream);
    printf("Result: %s\n", result->text);

    // 7. 清理资源
    SherpaOnnxDestroyOfflineRecognizerResult(result);
    SherpaOnnxDestroyOfflineStream(stream);
    SherpaOnnxDestroyOfflineRecognizer(recognizer);
    SherpaOnnxDestroyWave(wave);

    return 0;
}
```

## 编译和运行

### 1. 使用 Makefile
```bash
cd c-api-examples
make

# 运行示例
./decode-file-c-api ./models/ ./test.wav
```

### 2. 手动编译（Linux/macOS）
```bash
gcc -o decode-file-c-api decode-file-c-api.c \
    -I../sherpa-onnx/c-api \
    -L../build/lib \
    -lsherpa-onnx \
    -std=c99
```

### 3. Windows 编译
```cmd
cl /I..\sherpa-onnx\c-api ^
   decode-file-c-api.c ^
   /link /LIBPATH:..\build\lib ^
   sherpa-onnx.lib
```

## 高级用法

### 1. 实时处理示例
```c
// 伪代码：从麦克风实时识别
void realtime_recognition() {
    SherpaOnnxOnlineRecognizer *recognizer =
        SherpaOnnxCreateOnlineRecognizer(&online_config);

    SherpaOnnxOnlineStream *stream =
        SherpaOnnxCreateOnlineStream(recognizer);

    // 音频循环
    while (recording) {
        // 获取音频数据
        float *samples = get_audio_samples(&num_samples);

        // 提交给识别器
        SherpaOnnxAcceptWaveform(stream, 16000, samples, num_samples);

        // 检查端点
        if (SherpaOnnxIsEndpoint(stream)) {
            SherpaOnnxFinalizeStream(stream);
            SherpaOnnxResult *result = SherpaOnnxGetResult(stream);
            printf("Final: %s\n", result->text);
            SherpaOnnxDestroyResult(result);
            SherpaOnnxReset(stream);
        }
    }
}
```

### 2. 多线程处理
```c
// 工作线程处理
void *worker_thread(void *arg) {
    SherpaOnnxOfflineRecognizer *recognizer =
        SherpaOnnxCreateOfflineRecognizer(&config);

    while (1) {
        Task *task = get_task_from_queue();
        if (!task) break;

        // 处理音频
        SherpaOnnxOfflineStream *stream =
            SherpaOnnxCreateOfflineStream(recognizer);
        SherpaOnnxAcceptWaveformOffline(stream,
                                       task->sample_rate,
                                       task->samples,
                                       task->num_samples);

        SherpaOnnxDecodeOfflineStream(recognizer, stream);
        SherpaOnnxResult *result =
            SherpaOnnxGetOfflineStreamResult(stream);

        // 返回结果
        send_result_to_main_thread(result);

        // 清理
        SherpaOnnxDestroyOfflineStream(stream);
        SherpaOnnxDestroyOfflineRecognizerResult(result);
    }

    SherpaOnnxDestroyOfflineRecognizer(recognizer);
    return NULL;
}
```

### 3. 错误处理
```c
// 完整的错误处理
int safe_recognition(const char *model_path, const char *wav_path) {
    SherpaOnnxOfflineRecognizer *recognizer = NULL;
    SherpaOnnxWave *wave = NULL;
    SherpaOnnxOfflineStream *stream = NULL;
    int ret = -1;

    // 创建识别器
    SherpaOnnxOfflineRecognizerConfig config = {0};
    // ... 设置配置 ...

    recognizer = SherpaOnnxCreateOfflineRecognizer(&config);
    if (!recognizer) {
        fprintf(stderr, "Failed to create recognizer\n");
        goto cleanup;
    }

    // 读取音频
    wave = SherpaOnnxReadWave(wav_path);
    if (!wave) {
        fprintf(stderr, "Failed to read wave: %s\n", wav_path);
        goto cleanup;
    }

    // ... 识别逻辑 ...

    ret = 0;  // 成功

cleanup:
    if (recognizer) SherpaOnnxDestroyOfflineRecognizer(recognizer);
    if (wave) SherpaOnnxDestroyWave(wave);
    if (stream) SherpaOnnxDestroyOfflineStream(stream);

    return ret;
}
```

## 性能优化技巧

### 1. 内存管理
```c
// 重用识别器（避免重复创建）
static SherpaOnnxOfflineRecognizer *g_recognizer = NULL;

int recognize_file(const char *wav_file) {
    if (!g_recognizer) {
        g_recognizer = SherpaOnnxCreateOfflineRecognizer(&config);
    }

    // 使用已有的识别器
    SherpaOnnxOfflineStream *stream =
        SherpaOnnxCreateOfflineStream(g_recognizer);
    // ... 处理 ...
}
```

### 2. 批处理
```c
// 批量处理多个文件
void batch_process(const char **wav_files, int count) {
    SherpaOnnxOfflineRecognizer *recognizer =
        SherpaOnnxCreateOfflineRecognizer(&config);

    for (int i = 0; i < count; i++) {
        SherpaOnnxOfflineStream *stream =
            SherpaOnnxCreateOfflineStream(recognizer);

        // 并行处理（如果支持）
        #pragma omp parallel for
        // ... 处理音频 ...
    }
}
```

### 3. 配置优化
```c
// 根据硬件优化配置
SherpaOnnxOfflineRecognizerConfig get_optimal_config() {
    SherpaOnnxOfflineRecognizerConfig config = {0};

    // 获取 CPU 核心数
    int num_cores = get_cpu_cores();
    config.num_threads = num_cores > 0 ? num_cores : 1;

    // 启用内存映射（大文件）
    config.model_config.use_memory_map = 1;

    // 选择解码方法
    config.decoding_method =
        is_low_power_device() ? "greedy_search" : "modified_beam_search";

    return config;
}
```

## 移植指南

### 1. 嵌入式设备
```c
// ARM Cortex-A7 优化
#ifdef __ARM_ARCH
    // 使用 NEON 指令
    config.use_nnapi = 1;  // Android
    config.use_acl = 1;    // ARM Compute Library
#endif

// 内存受限设备
config.max_batch_size = 1;  // 减少内存使用
config.model_config.num_threads = 1;
```

### 2. 实时系统
```c
// 设置实时优先级
#ifdef __linux__
    struct sched_param param = {0};
    param.sched_priority = 80;
    sched_setscheduler(0, SCHED_FIFO, &param);
#endif

// 预分配内存
void init_memory_pool() {
    // 预分配缓冲区
    g_audio_buffer = malloc(MAX_AUDIO_SIZE);
    // ... 其他预分配 ...
}
```

## 常见问题

### Q: 如何处理不同采样率的音频？
A: 使用重采样或配置特征提取器：
```c
config.feat_config.sampling_rate = 16000;  // 模型期望的采样率
config.feat_config.feature_dim = 80;

// 如果音频采样率不同，需要重采样
```

### Q: 如何减少延迟？
A: 优化策略：
1. 使用流式模型
2. 减小 chunk size
3. 优化缓冲区大小
4. 使用更小的模型

### Q: 如何处理长音频？
A: 分段处理策略：
1. 使用 VAD 检测语音段
2. 按固定时长分段
3. 使用流式处理

## 变更记录

### 2025-12-10 07:44:45
- 📝 创建 C API 示例文档
- 🔧 分类整理示例代码
- 💻 添加核心示例解析
- ⚡ 记录优化和移植指南

---

*相关文件：[C API 接口](../sherpa-onnx/c-api/CLAUDE.md) | [C++ 核心](../sherpa-onnx/csrc/CLAUDE.md) | [Python 示例](../python-api-examples/CLAUDE.md)*