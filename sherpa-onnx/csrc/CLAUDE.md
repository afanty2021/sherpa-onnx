[根目录](../../CLAUDE.md) > [sherpa-onnx](../) > **csrc**

# Sherpa-ONNX C++ 核心实现

> 更新时间：2025-12-10 07:44:45

## 模块职责

这是 Sherpa-ONNX 的核心 C++ 实现模块，包含所有语音处理功能的基础实现。它提供了：

1. **语音识别引擎**：流式和非流式 ASR 实现
2. **语音合成引擎**：多种 TTS 模型支持
3. **说话人处理**：识别、日志、验证功能
4. **音频处理**：VAD、增强、分离等
5. **ONNX 模型推理**：统一的模型加载和推理接口

## 目录结构

```
csrc/
├── README.md                   # 核心示例说明
├── CMakeLists.txt              # C++ 库构建配置
├── *.h, *.cc                   # 核心头文件和实现
├── [feature]-model-config.h    # 各种模型的配置类
├── [feature]-model.h           # 各种模型的实现类
├── [feature]-impl.h            # 各种功能的具体实现
└── third-party/                # 第三方集成
    ├── ascend/                 # 华为昇腾 NPU 支持
    ├── axera/                  # Axera NPU 支持
    ├── axcl/                   # 华为 Atlas 支持
    └── qnn/                    # 高通 QNN 支持
```

## 核心组件

### 1. 模型配置类
- `offline-model-config.h/cc`：离线模型配置
- `online-model-config.h/cc`：在线模型配置
- `offline-tts-model-config.h/cc`：TTS 模型配置
- `offline-speaker-diarization-model-config.h/cc`：说话人日志模型配置

### 2. 识别器实现
- `offline-recognizer.h/cc`：离线识别器
- `online-recognizer.h/cc`：在线识别器
- `streaming-recognizer-impl.h`：流式识别器实现

### 3. TTS 实现
- `offline-tts.h/cc`：离线 TTS 实现
- `vits-model.h/cc`：VITS 模型
- `matcha-tts-model.h/cc`：Matcha TTS 模型
- `hifigan-vocoder.h/cc`：HiFiGAN 声码器

### 4. 说话人处理
- `speaker-embedding-extractor.h/cc`：说话人嵌入提取
- `speaker-embedding-manager.h/cc`：说话人嵌入管理
- `fast-clustering.h/cc`：快速聚类算法
- `offline-speaker-diarization.h/cc`：说话人日志

### 5. 音频处理
- `vad-model.h/cc`：语音活动检测
- `silero-vad-model.h/cc`：Silero VAD 模型
- `offline-speech-enhancement.h/cc`：语音增强
- `offline-source-separation.h/cc`：音源分离

## 关键模型支持

### ASR 模型
- **Paraformer**：阿里巴巴的非自回归模型
- **Zipformer**：Kaldi 的流式/非流式模型
- **Whisper**：OpenAI 的多语言模型
- **SenseVoice**：支持多语言和方言的模型
- **TeleSpeech**：科大讯飞 CTC 模型
- **NeMo**：NVIDIA 的模型系列

### TTS 模型
- **VITS**：变分推理文本到语音
- **Piper**：快速轻量 TTS
- **Matcha-TTS**：非自回归 TTS
- **Kokoro**：多语言情感 TTS
- **Coqui TTS**：开源 TTS 集成
- **VITS-melo**：中英文混合 TTS

### NPU 加速
- **RKNN**：瑞芯微 NPU 支持
- **QNN**：高通 Hexagon DSP/NPU
- **Ascend**：华为昇腾 NPU
- **Axera**：爱芯元智 NPU
- **Axcl**：华为 Atlas 计算架构

## API 设计模式

### 1. 工厂模式
```cpp
// 创建识别器
auto recognizer = OfflineRecognizer::Create(config);

// 创建 TTS
auto tts = OfflineTts::Create(config);
```

### 2. 配置驱动
```cpp
// 统一的配置结构
struct FeatureConfig {
  ModelConfig model;
  DecoderConfig decoder;
  // ...
};
```

### 3. 流式接口
```cpp
// 流式处理
recognizer->AcceptWaveform(samples);
auto result = recognizer->GetResult();
```

## 性能优化

### 1. 内存管理
- 使用对象池减少内存分配
- 延迟加载模型和资源
- 智能指针管理资源生命周期

### 2. 并发处理
- 模型推理线程池
- 音频处理流水线
- 无锁数据结构

### 3. 算法优化
- 量化和定点化
- 模型剪枝和蒸馏
- 自适应批处理

## 测试策略

### 单元测试
- 每个模型类型都有对应测试
- 使用模拟数据进行测试
- 边界条件测试

### 集成测试
- 端到端测试流程
- 实时性能测试
- 内存泄漏检测

### 性能基准
- RTF（实时因子）测试
- 吞吐量测试
- 准确率测试

## 常见问题

### Q: 如何添加新的模型支持？
A: 需要实现：
1. 模型配置类（继承自基础配置）
2. 模型实现类（继承自基础模型）
3. 在识别器中集成模型
4. 添加配置解析逻辑

### Q: 如何优化推理速度？
A: 优化策略：
1. 使用量化模型
2. 启用 NPU 加速
3. 调整批大小
4. 使用模型缓存

### Q: 如何处理长音频？
A: 处理方式：
1. 分段处理（VAD+分段）
2. 流式处理（实时）
3. 使用分块算法
4. 内存映射大文件

## 变更记录

### 2025-12-10 07:44:45
- 📝 创建模块文档
- 🏗️ 整理核心组件说明
- 🔧 记录 API 设计模式
- ⚡ 添加性能优化建议

---

*相关文件：[C API](../c-api/CLAUDE.md) | [Python 绑定](../python/CLAUDE.md) | [示例代码](../../c-api-examples/CLAUDE.md)*