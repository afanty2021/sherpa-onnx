[根目录](../../CLAUDE.md) > [sherpa-onnx](../) > **c-api**

# Sherpa-ONNX C API

> 更新时间：2025-12-10 07:44:45

## 模块职责

C API 模块为 Sherpa-ONNX 提供了跨语言调用的 C 接口层。这个模块的作用是：

1. **跨语言桥接**：为 Python、Java、C#、Go 等语言提供稳定的 C 接口
2. **ABI 稳定性**：保证二进制接口的向后兼容
3. **资源管理**：提供明确的资源生命周期管理
4. **错误处理**：统一的错误码和错误信息机制

## API 设计原则

### 1. 面向对象风格
```c
// 创建对象
SherpaOnnxRecognizer* recognizer = CreateOfflineRecognizer(config);

// 使用对象
SherpaOnnxResult* result = Recognize(recognizer, audio_data, num_samples);

// 销毁对象
DestroyOfflineRecognizer(recognizer);
DestroyResult(result);
```

### 2. 资源所有权明确
- 调用者负责释放返回的对象
- 创建函数返回新对象
- 销毁函数释放资源

### 3. 错误处理
```c
// 函数返回错误码
int ret = RecognizeFile(recognizer, filename);
if (ret != 0) {
    const char* error = GetErrorString(ret);
    // 处理错误
}
```

## 核心数据结构

### 1. 配置结构
```c
// 主配置
typedef struct SherpaOnnxOnlineRecognizerConfig {
  SherpaOnnxFeatureConfig feat_config;
  SherpaOnnxOnlineModelConfig model_config;
  // ...
} SherpaOnnxOnlineRecognizerConfig;

// 模型配置
typedef struct SherpaOnnxOnlineModelConfig {
  const char* zipformer;
  const char* paraformer;
  const char* tokens;
  int32_t num_threads;
  bool debug;
  const char* provider;
  // ...
} SherpaOnnxOnlineModelConfig;
```

### 2. 结果结构
```c
// 识别结果
typedef struct SherpaOnnxResult {
  const char* text;
  const char** tokens;
  int32_t num_tokens;
  float** timestamps;  // [num_tokens][1]
  int32_t num_timestamps;
} SherpaOnnxResult;
```

## 主要 API 函数

### 1. 创建和销毁
```c
// 在线识别器
SherpaOnnxOnlineRecognizer* CreateOnlineRecognizer(
    const SherpaOnnxOnlineRecognizerConfig* config);

// 离线识别器
SherpaOnnxOfflineRecognizer* CreateOfflineRecognizer(
    const SherpaOnnxOfflineRecognizerConfig* config);

// TTS
SherpaOnnxOfflineTts* CreateOfflineTts(
    const SherpaOnnxOfflineTtsConfig* config);

// 说话人日志
SherpaOnnxOfflineSpeakerDiarization* CreateOfflineSpeakerDiarization(
    const SherpaOnnxOfflineSpeakerDiarizationConfig* config);
```

### 2. 识别接口
```c
// 在线识别
void AcceptWaveform(SherpaOnnxOnlineRecognizer* recognizer,
                   const float* samples, int32_t num_samples);
int32_t IsEndpoint(SherpaOnnxOnlineRecognizer* recognizer);
void Reset(SherpaOnnxOnlineRecognizer* recognizer);
SherpaOnnxResult* GetResult(SherpaOnnxOnlineRecognizer* recognizer);

// 离线识别
SherpaOnnxResult* OfflineRecognize(SherpaOnnxOfflineRecognizer* recognizer,
                                  const float* samples, int32_t num_samples);
```

### 3. TTS 接口
```c
SherpaOnnxGeneratedAudio* Generate(SherpaOnnxOfflineTts* tts,
                                  const char* text,
                                  float speed, int32_t speaker);
```

### 4. 工具函数
```c
// 版本信息
const char* SherpaOnnxVersion();

// 错误处理
const char* SherpaOnnxGetErrorString(int32_t err_code);

// 配置验证
int32_t ValidateOnlineRecognizerConfig(
    const SherpaOnnxOnlineRecognizerConfig* config);
```

## 使用示例

### C 示例（离线识别）
```c
#include "sherpa-onnx/c-api.h"

int main() {
    // 1. 创建配置
    SherpaOnnxOfflineRecognizerConfig config;
    memset(&config, 0, sizeof(config));
    config.model_config.transducer.model = "./model.onnx";
    config.model_config.tokens = "./tokens.txt";

    // 2. 创建识别器
    SherpaOnnxOfflineRecognizer* recognizer =
        CreateOfflineRecognizer(&config);

    // 3. 读取音频
    float* audio = ReadWav("test.wav", &num_samples);

    // 4. 识别
    SherpaOnnxResult* result =
        OfflineRecognize(recognizer, audio, num_samples);

    // 5. 输出结果
    printf("Text: %s\n", result->text);

    // 6. 清理
    DestroyOfflineRecognizer(recognizer);
    DestroyResult(result);
    free(audio);

    return 0;
}
```

### Python 示例（通过 CFFI）
```python
from cffi import FFI

ffi = FFI()
ffi.cdef("""
    typedef struct SherpaOnnxRecognizerConfig ...;

    SherpaOnnxOfflineRecognizer* CreateOfflineRecognizer(
        const SherpaOnnxOfflineRecognizerConfig* config);

    SherpaOnnxResult* OfflineRecognize(
        SherpaOnnxOfflineRecognizer* recognizer,
        const float* samples, int32_t num_samples);
""")

# 加载库
lib = ffi.dlopen("./libsherpa-onnx.so")

# 使用 API
config = ffi.new("SherpaOnnxOfflineRecognizerConfig*")
# 设置配置...
recognizer = lib.CreateOfflineRecognizer(config)
# 识别...
```

## 内存管理

### 1. 对象生命周期
- 所有 Create* 函数返回的对象都需要对应的 Destroy*
- Result 对象在使用后必须销毁
- 字符串字面量（text, tokens）由库管理，不需要释放

### 2. 线程安全
- 不同的识别器实例可以在不同线程使用
- 同一个识别器实例不是线程安全的
- 全局函数（如 GetVersion）是线程安全的

### 3. 错误处理
- 函数返回 NULL/0 表示失败
- 使用 GetLastError 获取错误信息
- 错误信息在下次调用前有效

## 平台差异

### Windows
```c
// 导出宏
#ifdef SHERPA_ONNX_BUILD_SHARED
    #ifdef SHERPA_ONNX_EXPORTS
        #define SHERPA_ONNX_API __declspec(dllexport)
    #else
        #define SHERPA_ONNX_API __declspec(dllimport)
    #endif
#else
    #define SHERPA_ONNX_API
#endif
```

### Linux/macOS
```c
// 使用 GCC 属性
#define SHERPA_ONNX_API __attribute__((visibility("default")))
```

## 版本兼容性

### 语义化版本
- 主版本：不兼容的 API 修改
- 次版本：向后兼容的功能性新增
- 修订版本：向后兼容的问题修正

### ABI 稳定性
- 添加新的函数是安全的
- 添加新的结构体字段是安全的（在末尾）
- 修改函数签名或删除字段需要主版本升级

## 变更记录

### 2025-12-10 07:44:45
- 📝 创建 C API 文档
- 🏗️ 记录 API 设计原则
- 💻 添加使用示例
- 🔧 说明内存管理规则

---

*相关文件：[C++ 核心](../csrc/CLAUDE.md) | [Python 绑定](../python/CLAUDE.md) | [C API 示例](../../c-api-examples/CLAUDE.md)*