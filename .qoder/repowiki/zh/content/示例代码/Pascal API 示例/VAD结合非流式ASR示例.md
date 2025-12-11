# VAD结合非流式ASR示例

<cite>
**本文档中引用的文件**   
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas)
- [vad_with_whisper.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_whisper.pas)
- [vad_with_zipformer_ctc.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_zipformer_ctc.pas)
- [sherpa-onnx-vad-with-offline-asr.cc](file://sherpa-onnx/csrc/sherpa-onnx-vad-with-offline-asr.cc)
- [sherpa-onnx-vad-microphone-offline-asr.cc](file://sherpa-onnx/csrc/sherpa-onnx-vad-microphone-offline-asr.cc)
- [vad-with-non-streaming-asr.py](file://python-api-examples/vad-with-non-streaming-asr.py)
- [voice-activity-detector.h](file://sherpa-onnx/csrc/voice-activity-detector.h)
- [VadModelConfig.h](file://sherpa-onnx/csrc/vad-model-config.h)
</cite>

## 目录
1. [简介](#简介)
2. [集成模式分析](#集成模式分析)
3. [核心组件与配置](#核心组件与配置)
4. [端到端处理流程](#端到端处理流程)
5. [时间戳对齐与同步](#时间戳对齐与同步)
6. [优势与性能考量](#优势与性能考量)
7. [代码实现解析](#代码实现解析)
8. [结论](#结论)

## 简介

本文档详细阐述了在sherpa-onnx的Pascal API中，如何将语音活动检测（VAD）与非流式自动语音识别（ASR）模型相结合的使用方法。通过系统性地分析`vad_with_dolphin`、`vad_with_moonshine`、`vad_with_sense_voice`等示例，本文揭示了其背后的集成模式。该架构的核心思想是先利用VAD精确地检测出音频中的语音活动区域（即说话片段），然后仅将这些被识别为“语音”的片段送入计算成本更高的非流式ASR模型进行精确的文本转录。这种级联处理方式不仅能显著减少ASR模型的计算量，还能有效提高最终的识别准确率。

## 集成模式分析

sherpa-onnx中VAD与非流式ASR的集成遵循一种清晰的级联（Cascade）模式。该模式可以分解为两个主要阶段：**检测阶段**和**识别阶段**。

在**检测阶段**，一个轻量级的VAD模型（如Silero VAD）被用来实时或离线地扫描输入的音频流。VAD模型以固定大小的窗口（例如512个采样点）为单位处理音频，持续输出当前窗口是否包含语音的判断。当VAD检测到一段连续的语音活动时，它会将这段音频作为一个独立的“语音段”（Speech Segment）缓存起来。这些语音段包含了原始的音频采样数据以及该段在原始音频中的起始位置（以采样点计）。

在**识别阶段**，系统会遍历VAD输出的所有语音段。对于每一个语音段，都会创建一个非流式ASR模型的识别流（Stream），并将该语音段的完整音频数据送入此流中。非流式ASR模型（如Dolphin、Moonshine、SenseVoice、Whisper等）利用其强大的上下文理解能力，对该段完整的语音进行端到端的解码，生成最可能的文本转录结果。处理完一个语音段后，系统会立即输出该段的识别结果，包括文本内容和对应的时间戳。

这种模式的关键在于将“粗粒度的语音/非语音分割”任务交给高效的VAD模型，而将“细粒度的精确文本识别”任务留给功能强大但计算密集的ASR模型，从而实现了效率与精度的最佳平衡。

**Section sources**
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas#L1-L136)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas#L1-L140)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas#L1-L138)
- [sherpa-onnx-vad-with-offline-asr.cc](file://sherpa-onnx/csrc/sherpa-onnx-vad-with-offline-asr.cc#L132-L178)

## 核心组件与配置

该集成架构依赖于两个核心组件：`VoiceActivityDetector`（语音活动检测器）和`OfflineRecognizer`（离线识别器），它们的配置是成功集成的关键。

### VAD模型配置

VAD模型的配置主要通过`VadModelConfig`结构体进行。在Pascal API中，这体现为`TSherpaOnnxVadModelConfig`记录。其核心配置项包括：
- **`SileroVad.Model`**: 指向Silero VAD模型文件（`.onnx`）的路径。
- **`SileroVad.WindowSize`**: VAD模型处理音频的窗口大小，通常为512个采样点。
- **`SileroVad.Threshold`**: 语音检测的阈值，值越高越严格。
- **`SileroVad.MinSpeechDuration`**: 一个语音段被认定为有效所需的最短持续时间（秒）。
- **`SileroVad.MinSilenceDuration`**: 语音段之间静音间隔的最短持续时间（秒），用于合并相邻的语音段。
- **`SampleRate`**: 音频的采样率，必须与输入音频和ASR模型的要求一致，通常为16000 Hz。

### 非流式ASR模型配置

非流式ASR模型的配置则通过`OfflineRecognizerConfig`进行。在Pascal API中，这体现为`TSherpaOnnxOfflineRecognizerConfig`记录。其配置取决于所使用的具体ASR模型：
- **Dolphin CTC**: 需要指定`ModelConfig.Dolphin.Model`和`ModelConfig.Tokens`。
- **Moonshine**: 需要指定`ModelConfig.Moonshine.Preprocessor`、`Encoder`、`UncachedDecoder`、`CachedDecoder`和`Tokens`。
- **SenseVoice**: 需要指定`ModelConfig.SenseVoice.Model`和`Tokens`。
- **Whisper**: 需要指定`ModelConfig.Whisper.Encoder`、`Decoder`和`Tokens`。
- **Zipformer CTC**: 需要指定`ModelConfig.ZipformerCtc.Model`和`Tokens`。

所有ASR模型配置都共享`Provider`（如'cpu'）、`NumThreads`和`Debug`等通用设置。

**Section sources**
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas#L19-L42)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas#L19-L42)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas#L19-L42)
- [vad_with_whisper.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_whisper.pas#L19-L42)
- [vad_with_zipformer_ctc.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_zipformer_ctc.pas#L19-L42)
- [VadModelConfig.h](file://sherpa-onnx/csrc/vad-model-config.h)

## 端到端处理流程

VAD与非流式ASR结合的端到端处理流程可以概括为以下几个步骤：

1.  **初始化**：创建并配置`VoiceActivityDetector`和`OfflineRecognizer`实例。
2.  **音频加载**：读取输入的音频文件（如WAV格式），获取其采样率和原始采样数据。
3.  **采样率校验**：确保音频的采样率与VAD配置的采样率一致。
4.  **分块处理**：将音频数据按VAD的`WindowSize`进行分块。
5.  **VAD检测**：将每个音频块送入`Vad.AcceptWaveform`方法。VAD内部会累积数据并检测语音活动。
6.  **语音段提取**：在每次送入一个音频块后，检查VAD的输出队列（通过`Vad.IsEmpty`判断）。如果队列非空，则使用`Vad.Front`获取最前端的语音段，并用`Vad.Pop`将其移出队列。
7.  **ASR识别**：为每个提取出的语音段创建一个`OfflineStream`，调用`Stream.AcceptWaveform`传入该语音段的完整音频数据，然后调用`Recognizer.Decode`和`Recognizer.GetResult`进行解码并获取识别结果。
8.  **结果输出**：将识别出的文本与该语音段的时间戳（通过`SpeechSegment.Start`计算）一并输出。
9.  **处理尾部数据**：在所有音频块处理完毕后，调用`Vad.Flush`以确保所有剩余的语音段都被检测到，并对这些尾部语音段重复步骤6-8。
10. **资源释放**：最后，释放`Recognizer`和`Vad`对象占用的资源。

```mermaid
flowchart TD
A[开始] --> B[初始化VAD和ASR]
B --> C[加载音频文件]
C --> D[校验采样率]
D --> E[按窗口分块音频]
E --> F[送入VAD.AcceptWaveform]
F --> G{VAD队列非空?}
G --> |是| H[获取语音段 Front()]
G --> |否| I{还有音频块?}
H --> J[创建ASR流]
J --> K[送入语音段 AcceptWaveform]
K --> L[解码 Decode]
L --> M[获取结果 GetResult]
M --> N[输出文本和时间戳]
N --> O[移除语音段 Pop()]
O --> G
I --> |是| F
I --> |否| P[调用Vad.Flush]
P --> Q{尾部队列非空?}
Q --> |是| H
Q --> |否| R[释放资源]
R --> S[结束]
```

**Diagram sources **
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas#L75-L135)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas#L79-L139)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas#L77-L137)

## 时间戳对齐与同步

在VAD与ASR的级联架构中，时间戳的对齐与同步至关重要，它确保了最终输出的文本结果能准确地对应到原始音频中的时间位置。

VAD模型在检测到一个语音段时，会记录该段在原始音频流中的**起始采样点索引**（`SpeechSegment.Start`）。这个索引是相对于整个音频流从头开始计算的。当ASR模型完成对一个语音段的识别后，系统需要将这个起始索引转换为人类可读的时间戳（秒）。

转换公式非常简单：
`开始时间 (秒) = SpeechSegment.Start / 音频采样率`

例如，如果一个语音段的`Start`值为16000，且音频采样率为16000 Hz，那么该段的开始时间就是1.0秒。该语音段的持续时间则通过其包含的采样点总数除以采样率得到。

这种基于采样点索引的同步机制保证了时间戳的精确性。无论VAD如何分割音频，ASR模型处理的都是带有原始时间坐标的完整片段，因此其输出结果的时间信息是准确无误的。在Pascal示例代码中，这一过程通过`Start := SpeechSegment.Start / Wave.SampleRate;`和`Duration := Length(SpeechSegment.Samples) / Wave.SampleRate;`来实现。

**Section sources**
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas#L104-L107)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas#L108-L111)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas#L106-L109)
- [voice-activity-detector.h](file://sherpa-onnx/csrc/voice-activity-detector.h#L14-L17)

## 优势与性能考量

将VAD与非流式ASR结合使用带来了显著的优势，但也需要考虑一些性能因素。

### 优势

1.  **计算效率提升**：这是最核心的优势。非流式ASR模型通常计算量巨大。通过VAD预先过滤掉静音和非语音部分，ASR模型只需处理占总时长一小部分的语音段，从而大幅降低了整体的计算开销和功耗，特别适合在资源受限的设备上运行。
2.  **识别准确率提高**：VAD有效地去除了背景噪声和静音对ASR模型的干扰。ASR模型可以专注于处理纯净的语音信号，这通常能带来更高的识别准确率，尤其是在嘈杂环境中。
3.  **结果结构化**：VAD的输出天然地将连续的语音分割成独立的段落。这使得最终的识别结果也以“段落”为单位，结构更加清晰，便于后续处理（如字幕生成、对话分析等）。
4.  **低延迟响应**：虽然ASR模型本身是非流式的，但整个系统可以在检测到一个语音段后就立即对其进行识别并输出结果，而无需等待整个长音频文件处理完毕，从而实现了较低的端到端延迟。

### 性能考量

1.  **VAD参数调优**：`MinSpeechDuration`和`MinSilenceDuration`等参数需要根据具体应用场景进行调整。过短的语音段可能包含不完整的词句，影响ASR效果；过长的静音间隔可能导致本应合并的句子被错误分割。
2.  **模型兼容性**：必须确保VAD和ASR模型的采样率完全匹配。如果音频采样率不匹配，需要进行重采样，这会增加额外的计算成本。
3.  **内存管理**：在处理长音频时，需要合理管理内存，避免因缓存大量音频数据而导致内存溢出。

**Section sources**
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas#L32-L35)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas#L32-L35)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas#L32-L35)

## 代码实现解析

通过对`pascal-api-examples/vad-with-non-streaming-asr/`目录下多个示例文件的分析，可以发现其代码实现具有高度的一致性和模块化设计。

所有示例都遵循相同的主程序结构：
1.  **定义创建函数**：`CreateVad()`函数负责初始化并返回一个配置好的`TSherpaOnnxVoiceActivityDetector`对象。`CreateOfflineRecognizer()`函数则根据不同的ASR模型类型，创建并返回相应的`TSherpaOnnxOfflineRecognizer`对象。
2.  **主流程控制**：主程序首先调用这两个创建函数，然后加载音频文件。
3.  **核心处理循环**：程序进入一个`while`循环，以`WindowSize`为步长遍历整个音频数据。在循环体内，音频块被送入VAD，随后通过一个嵌套的`while`循环不断检查并处理VAD输出的语音段。
4.  **ASR处理**：对于每一个语音段，程序创建一个新的`OfflineStream`，将语音段数据送入流中，执行解码，并获取和输出结果。
5.  **尾部处理**：主循环结束后，调用`Vad.Flush`并处理所有剩余的语音段。
6.  **资源清理**：最后，使用`FreeAndNil`释放所有创建的对象。

这种设计模式清晰地分离了配置、初始化和处理逻辑，使得代码易于理解和维护。不同示例之间的差异仅在于`CreateOfflineRecognizer()`函数中对ASR模型的具体配置，而VAD的集成逻辑完全相同，体现了良好的代码复用性。

**Section sources**
- [vad_with_dolphin.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_dolphin.pas#L11-L135)
- [vad_with_moonshine.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_moonshine.pas#L11-L139)
- [vad_with_sense_voice.pas](file://pascal-api-examples/vad-with-non-streaming-asr/vad_with_sense_voice.pas#L11-L137)

## 结论

在sherpa-onnx的Pascal API中，VAD与非流式ASR的结合是一种高效且实用的语音识别架构。通过`vad_with_dolphin`、`vad_with_moonshine`和`vad_with_sense_voice`等示例，我们可以看到一个清晰、一致的集成模式：利用轻量级的VAD模型进行语音活动检测和音频分割，再将分割出的语音段送入功能强大的非流式ASR模型进行精确识别。这种级联处理方式不仅显著降低了计算成本，还提高了识别准确率和结果的结构性。通过精确的时间戳对齐机制，系统能够输出与原始音频完美同步的文本结果。该模式为构建高效、准确的语音识别应用提供了一个优秀的实践范例。