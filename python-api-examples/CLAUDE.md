[根目录](../CLAUDE.md) > **python-api-examples**

# Python API 示例

> 更新时间：2025-12-10 07:44:45

## 模块职责

本模块包含 Sherpa-ONNX 的 Python API 使用示例，展示了如何通过 Python 调用各种语音处理功能。这些示例涵盖了：

1. **语音识别**：在线/离线、流式/非流式识别
2. **语音合成**：多种 TTS 模型使用
3. **说话人处理**：识别、日志、验证
4. **实时处理**：麦克风输入、WebSocket 服务
5. **高级应用**：字幕生成、服务器部署

## 示例分类

### 🎤 语音识别 (ASR)

#### 在线识别（流式）
- `online-decode-files.py` - 基础在线文件识别
- `streaming-paraformer-asr-microphone.py` - Paraformer 麦克风实时识别
- `streaming-zipformer-ctc-hlg-decode-file.py` - Zipformer CTC + HLG 解码
- `two-pass-speech-recognition-from-microphone.py` - 两遍识别（快速+精调）

#### 离线识别（非流式）
- `offline-decode-files.py` - 基础离线文件识别
- `offline-whisper-decode-files.py` - Whisper 模型使用
- `offline-sense-voice-ctc-decode-files.py` - SenseVoice 多语言识别
- `offline-zipformer-ctc-decode-files.py` - Zipformer CTC 模型
- `offline-moonshine-decode-files.py` - Moonshine 轻量模型

#### 特殊模型
- `offline-nemo-parakeet-decode-file.py` - NVIDIA NeMo Transducer
- `offline-dolphin-ctc-decode-files.py` - Dolphin 多语言模型
- `offline-fire-red-asr-decode-files.py` - Fire Red ASR
- `offline-telespeech-ctc-decode-files.py` - 科大讯飞 TeleSpeech
- `offline-omnilingual-asr-ctc-decode-files.py` - 1600种语言模型

### 🔊 语音合成 (TTS)

#### 基础 TTS
- `offline-tts.py` - 基础离线 TTS
- `offline-tts-play.py` - TTS 并播放
- `offline-zeroshot-tts.py` - 零样本声音克隆

#### 特定 TTS 模型
```python
# VITS 模型
vits_model = {
    "vits": "./vits-model.onnx",
    "lexicon": "./lexicon.txt",
}

# Matcha TTS
matcha_model = {
    "matcha": "./matcha-model.onnx",
}

# Kokoro 多语言
kokoro_model = {
    "kokoro": "./kokoro-model.onnx",
    "lexicon": "./kokoro-lexicon.txt",
}
```

### 👥 说话人处理

#### 说话人识别
- `speaker-identification.py` - 说话人识别基础
- `speaker-identification-with-vad.py` - 结合 VAD 的识别
- `speaker-identification-with-vad-dynamic.py` - 动态说话人识别

#### 说话人日志
- `offline-speaker-diarization.py` - 离线说话人日志
- `offline-speaker-diarization.py` 示例代码：
```python
import sherpa_onnx

# 配置说话人日志
config = sherpa_onnx.OfflineSpeakerDiarizationConfig(
    vad_config=vad_config,
    embedding_config=embedding_config,
    clustering_config=clustering_config,
)

# 创建处理实例
sd = sherpa_onnx.OfflineSpeakerDiarization(config)

# 处理音频
audio = sherpa_onnx.read_wav("audio.wav")
segments = sd.process(audio)

# 输出结果
for seg in segments.segments:
    print(f"时间: {seg.start:.2f} - {seg.end:.2f}")
    print(f"说话人: {seg.speaker}")
```

### 🌐 服务端应用

#### WebSocket 服务
- `streaming_server.py` - 流式识别服务器
- `non_streaming_server.py` - 非流式识别服务器
- `two-pass-wss.py` - 两遍识别 WebSocket 服务
- `http_server.py` - HTTP REST API 服务

#### WebSocket 客户端
- `online-websocket-client-decode-file.py` - 文件解码客户端
- `online-websocket-client-microphone.py` - 麦克风客户端
- `offline-websocket-client-decode-files-sequential.py` - 顺序解码

### 🎯 高级功能

#### VAD（语音活动检测）
- `vad-microphone.py` - 麦克风 VAD
- `vad-alsa.py` - Linux ALSA VAD
- `vad-with-non-streaming-asr.py` - VAD + ASR 组合

#### 语音增强
- `offline-speech-enhancement-gtcrn.py` - GT-CRN 语音增强
- `offline-source-separation-spleeter.py` - Spleeter 音源分离
- `offline-source-separation-uvr.py` - UVR 音源分离

#### 实用工具
- `generate-subtitles.py` - 字幕生成
- `add-punctuation.py` - 标点恢复
- `spoken-language-identification.py` - 语言识别

## 使用示例

### 1. 快速开始 - 语音识别

```python
import sherpa_onnx
import wave

# 1. 创建识别器
recognizer_config = sherpa_onnx.OfflineRecognizerConfig(
    feat_config=sherpa_onnx.FeatureConfig(
        sampling_rate=16000,
        feature_dim=80,
    ),
    model_config=sherpa_onnx.OfflineModelConfig(
        transducer=sherpa_onnx.OfflineTransducerModelConfig(
            model="./model.onnx",
        ),
        tokens="./tokens.txt",
        num_threads=2,
    ),
)

recognizer = sherpa_onnx.OfflineRecognizer(recognizer_config)

# 2. 读取音频
with wave.open("test.wav", "rb") as wf:
    audio = wf.readframes(-1)
    audio = np.frombuffer(audio, dtype=np.float32)

# 3. 识别
stream = recognizer.create_stream()
stream.accept_waveform(16000, audio)
recognizer.decode_stream(stream)

# 4. 获取结果
result = stream.result
print(f"识别结果: {result.text}")
```

### 2. 实时语音识别

```python
import sherpa_onnx
import pyaudio

# 配置
 recognizer_config = sherpa_onnx.OnlineRecognizerConfig(...)
 recognizer = sherpa_onnx.OnlineRecognizer(recognizer_config)

# 创建流
stream = recognizer.create_stream()

# 音频回调
def callback(in_data, frame_count, time_info, status):
    samples = np.frombuffer(in_data, dtype=np.float32)
    stream.accept_waveform(16000, samples)

    # 获取部分结果
    if recognizer.is_ready(stream):
        recognizer.decode_stream(stream)
        result = recognizer.get_result(stream)
        if result.text:
            print(f"部分结果: {result.text}")

    return (in_data, pyaudio.paContinue)

# 启动音频流
p = pyaudio.PyAudio()
stream_audio = p.open(
    format=pyaudio.paFloat32,
    channels=1,
    rate=16000,
    input=True,
    frames_per_buffer=1600,
    stream_callback=callback,
)

stream_audio.start_stream()
stream_audio.wait_for_completion()
```

### 3. TTS 语音合成

```python
import sherpa_onnx
import soundfile as sf

# 配置 TTS
tts_config = sherpa_onnx.OfflineTtsConfig(
    model=sherpa_onnx.OfflineTtsModelConfig(
        vits=sherpa_onnx.OfflineTtsVitsModelConfig(
            model="./vits.onnx",
            lexicon="./lexicon.txt",
        ),
        tokens="./tokens.txt",
        num_threads=2,
    ),
    rule_fsts="./rule.fst",
)

# 创建 TTS 实例
tts = sherpa_onnx.OfflineTts(tts_config)

# 生成语音
audio = tts.generate("你好，世界！")

# 保存
sf.write("output.wav", audio.samples, audio.sample_rate)

# 播放（可选）
tts.play(audio)
```

## 最佳实践

### 1. 性能优化
```python
# 使用模型缓存
recognizer = sherpa_onnx.OfflineRecognizer(config)
# 复用识别器处理多个文件

# 批处理
for audio_batch in audio_batches:
    results = recognizer.recognize_batch(audio_batch)
```

### 2. 错误处理
```python
try:
    recognizer = sherpa_onnx.OfflineRecognizer(config)
except RuntimeError as e:
    print(f"模型加载失败: {e}")
    sys.exit(1)
```

### 3. 资源管理
```python
# 使用 with 语句（如果支持）
with sherpa_onnx.OfflineRecognizer(config) as recognizer:
    result = recognizer.recognize(audio)

# 或手动释放
recognizer = sherpa_onnx.OfflineRecognizer(config)
try:
    # 使用 recognizer
    pass
finally:
    del recognizer
```

## 故障排除

### 常见问题

1. **模型加载失败**
   - 检查模型路径
   - 验证模型格式
   - 确认 ONNX Runtime 版本

2. **音频格式错误**
   - 确保采样率匹配
   - 检查音频数据类型（float32）
   - 验证通道数（单声道）

3. **性能问题**
   - 减少线程数
   - 使用量化模型
   - 启用 GPU 加速

### 调试技巧

```python
# 启用调试模式
config.model_config.debug = True

# 打印配置
print(recognizer_config)

# 检查模型信息
print(f"模型路径: {config.model_config.transducer.model}")
```

## 变更记录

### 2025-12-10 07:44:45
- 📝 创建 Python 示例文档
- 🏷️ 分类整理示例代码
- 💻 添加使用示例和最佳实践
- 🔧 记录故障排除指南

---

*相关文件：[Python 绑定](../sherpa-onnx/python/CLAUDE.md) | [C++ 核心](../sherpa-onnx/csrc/CLAUDE.md) | [C API 示例](../c-api-examples/CLAUDE.md)*