# T-One CTC流式语音识别示例

<cite>
**本文档引用文件**   
- [streaming-t-one-ctc-cxx-api.cc](file://cxx-api-examples/streaming-t-one-ctc-cxx-api.cc)
- [online-t-one-ctc-model-config.h](file://sherpa-onnx/csrc/online-t-one-ctc-model-config.h)
- [online-t-one-ctc-model.h](file://sherpa-onnx/csrc/online-t-one-ctc-model.h)
- [online-stream.h](file://sherpa-onnx/csrc/online-stream.h)
- [streaming-zipformer-cxx-api.cc](file://cxx-api-examples/streaming-zipformer-cxx-api.cc)
- [online-zipformer2-ctc-model.h](file://sherpa-onnx/csrc/online-zipformer2-ctc-model.h)
</cite>

## 目录
1. [引言](#引言)
2. [T-One CTC模型架构解析](#t-one-ctc模型架构解析)
3. [OnlineModelConfig配置差异分析](#onlinemodelconfig配置差异分析)
4. [OnlineStream接口与音频流处理](#onlinestream接口与音频流处理)
5. [端到端C++代码示例](#端到端c代码示例)
6. [性能表现与适用范围](#性能表现与适用范围)
7. [结论](#结论)

## 引言
T-One CTC模型是sherpa-onnx项目中一种高效的流式自动语音识别（ASR）模型，专为低延迟场景设计。本文档深入解析T-One CTC模型在流式ASR中的独特实现机制，重点说明其与Zipformer等其他流式模型在OnlineModelConfig配置上的差异。详细描述如何通过OnlineStream接口进行音频流的增量输入处理，包括AcceptWaveform的数据格式要求、IsReady的触发条件以及GetResult返回部分识别结果的策略。分析T-One CTC模型在低延迟场景下的性能表现和适用范围，并提供完整的端到端代码示例，涵盖模型加载、音频流处理、实时识别结果获取和系统资源释放等环节。

**Section sources**
- [streaming-t-one-ctc-cxx-api.cc](file://cxx-api-examples/streaming-t-one-ctc-cxx-api.cc)

## T-One CTC模型架构解析
T-One CTC模型基于连接时序分类（Connectionist Temporal Classification, CTC）算法，采用流式处理架构，能够实时处理音频流并输出部分识别结果。该模型通过将音频信号分割成小块（chunks）进行处理，每处理完一个chunk就更新识别结果，从而实现低延迟的流式识别。

T-One CTC模型的核心组件包括：
- **特征提取器**：将原始音频波形转换为适合模型处理的特征向量。
- **编码器**：对特征向量进行编码，提取高层次的语音特征。
- **CTC解码器**：利用CTC算法将编码后的特征序列映射到字符序列，生成最终的文本输出。

模型的流式处理能力得益于其对音频流的分块处理机制，每个chunk的处理独立且高效，确保了实时性。

**Section sources**
- [online-t-one-ctc-model.h](file://sherpa-onnx/csrc/online-t-one-ctc-model.h)
- [online-t-one-ctc-model-config.h](file://sherpa-onnx/csrc/online-t-one-ctc-model-config.h)

## OnlineModelConfig配置差异分析
T-One CTC模型与Zipformer等其他流式模型在OnlineModelConfig配置上存在显著差异。T-One CTC模型的配置主要集中在`OnlineToneCtcModelConfig`结构体中，而Zipformer模型则使用`OnlineTransducerModelConfig`或`OnlineZipformer2CtcModelConfig`。

### T-One CTC模型配置
```cpp
struct OnlineToneCtcModelConfig {
  std::string model;
};
```
- **model**：指定T-One CTC模型的ONNX文件路径。

### Zipformer模型配置
```cpp
struct OnlineTransducerModelConfig {
  std::string encoder;
  std::string decoder;
  std::string joiner;
};
```
- **encoder**：指定编码器模型的ONNX文件路径。
- **decoder**：指定解码器模型的ONNX文件路径。
- **joiner**：指定连接器模型的ONNX文件路径。

T-One CTC模型的配置更为简洁，仅需指定一个模型文件，而Zipformer模型需要分别指定编码器、解码器和连接器三个模型文件。这种差异反映了两种模型在架构上的不同：T-One CTC模型采用单一模型进行端到端的流式识别，而Zipformer模型则采用多组件协同工作的复杂架构。

**Section sources**
- [online-t-one-ctc-model-config.h](file://sherpa-onnx/csrc/online-t-one-ctc-model-config.h)
- [online-zipformer2-ctc-model.h](file://sherpa-onnx/csrc/online-zipformer2-ctc-model.h)

## OnlineStream接口与音频流处理
OnlineStream接口是T-One CTC模型进行流式音频处理的核心。通过该接口，可以实现音频流的增量输入处理，实时获取部分识别结果。

### AcceptWaveform数据格式要求
`AcceptWaveform`方法用于向OnlineStream输入音频数据，其参数要求如下：
- **sampling_rate**：音频的采样率，单位为Hz。
- **waveform**：指向音频样本数组的指针，样本值必须归一化到[-1, 1]范围内。
- **n**：音频样本的数量。

### IsReady触发条件
`IsReady`方法用于检查OnlineStream是否准备好进行解码。当输入的音频数据足够多，足以生成有意义的识别结果时，该方法返回true。通常，这需要至少一个chunk的音频数据。

### GetResult返回策略
`GetResult`方法用于获取当前的识别结果。在流式处理过程中，该方法会返回部分识别结果，随着更多音频数据的输入，结果会逐步完善。最终结果在所有音频数据处理完毕后返回。

**Section sources**
- [online-stream.h](file://sherpa-onnx/csrc/online-stream.h)
- [streaming-t-one-ctc-cxx-api.cc](file://cxx-api-examples/streaming-t-one-ctc-cxx-api.cc)

## 端到端C++代码示例
以下是一个完整的端到端C++代码示例，展示了如何使用T-One CTC模型进行流式语音识别：

```cpp
#include <iostream>
#include "sherpa-onnx/c-api/cxx-api.h"

int32_t main() {
  using namespace sherpa_onnx::cxx;
  OnlineRecognizerConfig config;

  config.model_config.t_one_ctc.model = "path/to/model.onnx";
  config.model_config.tokens = "path/to/tokens.txt";
  config.model_config.num_threads = 1;

  OnlineRecognizer recognizer = OnlineRecognizer::Create(config);
  if (!recognizer.Get()) {
    std::cerr << "Failed to load model\n";
    return -1;
  }

  Wave wave = ReadWave("path/to/audio.wav");
  OnlineStream stream = recognizer.CreateStream();
  stream.AcceptWaveform(wave.sample_rate, wave.samples.data(), wave.samples.size());
  stream.InputFinished();

  while (recognizer.IsReady(&stream)) {
    recognizer.Decode(&stream);
  }

  OnlineRecognizerResult result = recognizer.GetResult(&stream);
  std::cout << "Recognition result: " << result.text << "\n";

  return 0;
}
```

**Section sources**
- [streaming-t-one-ctc-cxx-api.cc](file://cxx-api-examples/streaming-t-one-ctc-cxx-api.cc)

## 性能表现与适用范围
T-One CTC模型在低延迟场景下表现出色，适用于实时语音识别应用，如语音助手、实时字幕生成等。其流式处理机制确保了识别结果的及时性，同时保持了较高的识别准确率。然而，由于CTC算法的特性，T-One CTC模型在处理长音频时可能会出现识别错误累积的问题，因此更适合短语音片段的识别任务。

**Section sources**
- [streaming-t-one-ctc-cxx-api.cc](file://cxx-api-examples/streaming-t-one-ctc-cxx-api.cc)

## 结论
T-One CTC模型作为一种高效的流式ASR模型，在低延迟场景下具有显著优势。通过深入理解其架构和配置差异，开发者可以更好地利用OnlineStream接口进行音频流的增量处理，实现实时语音识别。本文提供的端到端代码示例为开发者提供了实践指导，帮助其快速上手并应用T-One CTC模型于实际项目中。