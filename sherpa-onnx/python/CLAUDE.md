[根目录](../../CLAUDE.md) > [sherpa-onnx](../) > **python**

# Python 绑定

> 更新时间：2025-12-10 07:48:28

## 模块职责

Sherpa-ONNX 的 Python 绑定模块提供了将 C++ 核心功能暴露给 Python 语言的接口。通过 pybind11 框架，它实现了高效的无缝集成，让 Python 开发者能够轻松使用所有语音 AI 功能，包括：

- **语音识别 (ASR)**：在线/离线识别
- **语音合成 (TTS)**：多模型支持
- **说话人处理**：识别、日志、验证
- **语音增强**：降噪、音源分离
- **其他功能**：VAD、关键词检测、语言识别

## 入口与启动

### 主要绑定文件
```
sherpa-onnx/python/csrc/
├── offline-recognizer.cc     # 离线识别绑定
├── online-recognizer.cc      # 在线识别绑定
├── offline-tts.cc            # TTS 绑定
├── speaker-*.cc              # 说话人相关绑定
├── vad.cc                    # VAD 绑定
└── keyword-spotter.cc        # 关键词检测绑定
```

### 构建和安装
```bash
# 方法1：从源码构建
cd sherpa-onnx
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release \
      -DSHERPA_ONNX_ENABLE_PYTHON=ON \
      -DPython3_EXECUTABLE=$(which python3) \
      ..
make -j4

# 安装到 Python 环境
cd python
pip install -e .

# 方法2：使用 pip 安装预编译包
pip install sherpa-onnx
```

## 对外接口

### 1. 语音识别接口

#### 离线语音识别
```python
import sherpa_onnx

# 配置离线识别器
config = sherpa_onnx.OfflineRecognizerConfig(
    feat_config=sherpa_onnx.FeatureExtractorConfig(
        sampling_rate=16000,
        feature_dim=80,
    ),
    model_config=sherpa_onnx.OfflineModelConfig(
        transducer=sherpa_onnx.OfflineTransducerModelConfig(
            encoder="./encoder.onnx",
            decoder="./decoder.onnx",
            joiner="./joiner.onnx",
        ),
        tokens="./tokens.txt",
        num_threads=2,
        debug=False,
        provider="cpu",
    ),
    decoding_method="greedy_search",
    max_active_paths=4,
)

# 创建识别器
recognizer = sherpa_onnx.OfflineRecognizer(config)

# 创建音频流
stream = recognizer.create_stream()
stream.accept_waveform(16000, audio_samples)

# 执行识别
recognizer.decode_stream(stream)
result = stream.result

print(f"识别结果: {result.text}")
```

#### 在线语音识别
```python
# 配置在线识别器
online_config = sherpa_onnx.OnlineRecognizerConfig(
    feat_config=sherpa_onnx.FeatureExtractorConfig(
        sampling_rate=16000,
        feature_dim=80,
    ),
    model_config=sherpa_onnx.OnlineModelConfig(
        zipformer=sherpa_onnx.OnlineZipformerModelConfig(
            encoder="./encoder.onnx",
            decoder="./decoder.onnx",
            joiner="./joiner.onnx",
        ),
        tokens="./tokens.txt",
        num_threads=2,
    ),
    endpoint_config=sherpa_onnx.EndpointConfig(),
    lm_config=sherpa_onnx.OnlineLMConfig(),
)

# 创建识别器
online_recognizer = sherpa_onnx.OnlineRecognizer(online_config)

# 创建流
stream = online_recognizer.create_stream()

# 实时处理音频
def process_audio_chunk(audio_chunk):
    stream.accept_waveform(16000, audio_chunk)

    # 获取部分结果
    if online_recognizer.is_ready(stream):
        online_recognizer.decode_stream(stream)
        result = online_recognizer.get_result(stream)
        if result.text:
            print(f"部分结果: {result.text}")
```

### 2. 语音合成接口

```python
# 配置 TTS
tts_config = sherpa_onnx.OfflineTtsConfig(
    model=sherpa_onnx.OfflineTtsModelConfig(
        vits=sherpa_onnx.OfflineTtsVitsModelConfig(
            model="./vits.onnx",
            lexicon="./lexicon.txt",
            tokens="./tokens.txt",
        ),
        num_threads=2,
    ),
    rule_fsts="./rule.fst",
    max_num_sentences=1,
)

# 创建 TTS 实例
tts = sherpa_onnx.OfflineTts(tts_config)

# 生成语音
audio = tts.generate(
    text="你好，世界！",
    speaker_id=0,
    speed=1.0,
)

# audio 包含:
# - audio.samples: 音频数据 (numpy array)
# - audio.sample_rate: 采样率
```

### 3. 说话人识别接口

```python
# 配置说话人识别
speaker_config = sherpa_onnx.SpeakerEmbeddingExtractorConfig(
    model="./speaker_model.onnx",
    num_threads=2,
)

# 创建提取器
extractor = sherpa_onnx.SpeakerEmbeddingExtractor(speaker_config)

# 提取说话人特征
stream = extractor.create_stream()
stream.accept_waveform(16000, audio_samples)
embedding = extractor.compute(stream)

# 说话人验证
verifier_config = sherpa_onnx.SpeakerEmbeddingVerifierConfig(
    extractor_config=speaker_config,
    threshold=0.5,
)
verifier = sherpa_onnx.SpeakerEmbeddingVerifier(verifier_config)

# 验证说话人
score = verifier.verify(embedding1, embedding2)
print(f"相似度分数: {score}")
```

### 4. VAD 接口

```python
# 配置 VAD
vad_config = sherpa_onnx.VadModelConfig(
    silero_vad=sherpa_onnx.SileroVadModelConfig(
        model="./silero_vad.onnx",
        threshold=0.5,
        min_silence_duration=0.5,
        min_speech_duration=0.25,
    ),
    sample_rate=16000,
    window_size=512,
)

# 创建 VAD 实例
vad = sherpa_onnx.VoiceActivityDetector(vad_config)

# 处理音频
vad.accept_waveform(audio_chunk)
if vad.is_speech_detected():
    print("检测到语音")
```

## 关键依赖与配置

### 核心依赖
- **pybind11**: C++/Python 绑定框架
- **ONNX Runtime**: 模型推理引擎
- **NumPy**: 音频数据处理
- **librosa** (可选): 音频分析

### 配置类层次结构

```
配置类
├── FeatureExtractorConfig      # 特征提取配置
├── OfflineModelConfig          # 离线模型配置
│   ├── OfflineTransducerModelConfig
│   ├── OfflineParaformerModelConfig
│   ├── OfflineWhisperModelConfig
│   └── ...
├── OnlineModelConfig           # 在线模型配置
│   ├── OnlineZipformerModelConfig
│   ├── OnlineParaformerModelConfig
│   └── ...
├── OfflineTtsModelConfig       # TTS 模型配置
│   ├── OfflineTtsVitsModelConfig
│   ├── OfflineTtsMatchaModelConfig
│   └── ...
├── SpeakerEmbeddingExtractorConfig  # 说话人模型配置
└── VadModelConfig              # VAD 模型配置
```

## 数据模型

### 主要数据结构

#### 识别结果
```python
class OfflineRecognitionResult:
    text: str                 # 识别文本
    tokens: List[str]         # Token 序列
    timestamps: List[float]   # 时间戳（可选）
```

#### TTS 音频
```python
class GeneratedAudio:
    samples: np.ndarray       # 音频数据 (float32)
    sample_rate: int          # 采样率
```

#### 说话人分割结果
```python
class SpeakerDiarizationResult:
    segments: List[Segment]   # 语音段

class Segment:
    start: float              # 开始时间（秒）
    end: float                # 结束时间（秒）
    speaker: int              # 说话人 ID
```

## 测试与质量

### 单元测试
```python
# 运行测试
python -m pytest tests/

# 或使用 unittest
python -m unittest discover tests/
```

### 性能测试
```python
import time

# 测试识别速度
start_time = time.time()
result = recognizer.recognize(audio)
duration = time.time() - start_time

rtf = duration / (len(audio) / sample_rate)
print(f"实时因子 (RTF): {rtf:.2f}")
```

## 常见问题 (FAQ)

### Q1: 如何处理不同的音频格式？
A: 使用 librosa 或 soundfile 转换：
```python
import soundfile as sf

# 读取音频
audio, sr = sf.read("input.wav")
if sr != 16000:
    # 重采样
    import librosa
    audio = librosa.resample(audio, orig_sr=sr, target_sr=16000)
```

### Q2: 如何减少内存使用？
A:
1. 使用量化模型
2. 减少线程数
3. 及时释放音频数据
```python
# 使用完后清理
del stream
del audio_chunk
```

### Q3: 如何提高处理速度？
A:
1. 使用 GPU 加速：
```python
config.model_config.provider = "cuda"
```
2. 增加线程数（在合理范围内）
3. 使用更小的模型

### Q4: 批处理如何实现？
A: Python 绑定本身不支持批处理，但可以在 Python 层实现：
```python
def batch_recognize(audio_list, recognizer):
    results = []
    for audio in audio_list:
        stream = recognizer.create_stream()
        stream.accept_waveform(16000, audio)
        recognizer.decode_stream(stream)
        results.append(stream.result)
    return results
```

## 相关文件清单

### 核心绑定文件
- `csrc/offline-recognizer.cc` - 离线 ASR 绑定
- `csrc/online-recognizer.cc` - 在线 ASR 绑定
- `csrc/offline-tts.cc` - TTS 绑定
- `csrc/speaker-embedding-extractor.cc` - 说话人嵌入
- `csrc/vad.cc` - VAD 绑定

### 配置绑定文件
- `csrc/offline-model-config.cc` - 离线模型配置
- `csrc/online-model-config.cc` - 在线模型配置
- `csrc/offline-tts-model-config.cc` - TTS 模型配置

### 实用工具
- `csrc/alsa.cc` - Linux ALSA 音频接口
- `csrc/display.cc` - 实时显示
- `csrc/circular-buffer.cc` - 循环缓冲区

### 示例代码
见 `python-api-examples/` 目录

## 变更记录

### 2025-12-10 07:48:28
- 📝 创建 Python 绑定文档
- 🔗 添加 API 使用示例
- 💡 补充常见问题解答
- 📊 列出所有绑定接口

---

*相关文件：[Python 示例](../../python-api-examples/CLAUDE.md) | [C++ 核心](../csrc/CLAUDE.md) | [C API](../c-api/CLAUDE.md)*