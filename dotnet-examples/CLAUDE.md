# .NET API 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / .NET API 示例

## 模块概述

本模块包含 Sherpa-ONNX .NET (C#) API 的使用示例，展示了如何在 .NET 应用程序中集成离线语音处理功能。支持 .NET 6.0+，可在 Windows、Linux 和 macOS 上运行。

## 功能分类

### 🎤 语音识别 (ASR) 示例

#### 离线识别
- **offline-decode-files**: 批量文件语音识别
- **non-streaming-canary-decode-files**: NVIDIA NeMo Canary 模型

#### 关键词检测
- **keyword-spotting-from-files**: 从文件检测关键词
- **keyword-spotting-from-microphone**: 实时麦克风关键词检测

### 🔊 语音合成 (TTS) 示例

#### 离线 TTS
- **offline-tts**: 离线文本转语音
- **offline-tts-play**: 语音合成并播放

#### 特定模型
- **kitten-tts**: Kitten TTS 模型
- **kitten-tts-play**: Kitten TTS 并播放
- **kokoro-tts**: Kokoro 高质量 TTS
- **kokoro-tts-play**: Kokoro TTS 并播放

### 🎯 其他功能示例

#### 音频处理
- **offline-audio-tagging**: 音频事件检测
- **offline-speaker-diarization**: 说话人分离

#### 文本处理
- **offline-punctuation**: 标点符号恢复

## 快速开始

### 1. 环境要求
- .NET 6.0 或更高版本
- Windows/Linux/macOS
- ONNX Runtime 库

### 2. 安装 .NET
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y dotnet-sdk-6.0

# CentOS/RHEL
sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
sudo yum install -y dotnet-sdk-6.0

# macOS
brew install dotnet

# Windows
# 从 https://dotnet.microsoft.com/download 下载安装
```

### 3. 创建新项目
```bash
dotnet new console -n SherpaOnnxExample
cd SherpaOnnxExample
```

### 4. 添加 Sherpa-ONNX NuGet 包
```bash
dotnet add package SherpaOnnx
```

## 使用示例

### 基本语音识别

```csharp
// Program.cs
using SherpaOnnx;
using System;

class Program
{
    static void Main(string[] args)
    {
        // 配置模型
        var modelConfig = new OfflineModelConfig()
        {
            Transducer = new OfflineTransducerModelConfig()
            {
                Encoder = "./encoder.onnx",
                Decoder = "./decoder.onnx",
                Joiner = "./joiner.onnx"
            },
            Tokens = "./tokens.txt",
            NumThreads = 2,
            ModelType = "zipformer"
        };

        // 创建识别器
        using var recognizer = new OfflineRecognizer(modelConfig);

        // 识别音频文件
        var audioFile = "./test.wav";
        var waveData = WaveReader.Read(audioFile);

        var stream = recognizer.CreateStream();
        stream.AcceptWaveform(waveData.Samples, waveData.SampleRate);

        // 执行识别
        recognizer.Decode(stream);
        var result = recognizer.GetResult(stream);

        Console.WriteLine($"识别结果: {result.Text}");
    }
}
```

### 实时语音识别

```csharp
using SherpaOnnx;
using System;
using System.Threading;

class RealTimeAsr
{
    static void Main(string[] args)
    {
        // 配置在线模型
        var modelConfig = new OnlineModelConfig()
        {
            Transducer = new OnlineTransducerModelConfig()
            {
                Encoder = "./encoder.onnx",
                Decoder = "./decoder.onnx",
                Joiner = "./joiner.onnx"
            },
            Tokens = "./tokens.txt",
            NumThreads = 1
        };

        var recognizerConfig = new OnlineRecognizerConfig()
        {
            ModelConfig = modelConfig,
            EnableEndpoint = true,
            Rule1MinSilenceDuration = 0.5f
        };

        using var recognizer = new OnlineRecognizer(recognizerConfig);

        // 创建音频捕获器
        using var audioCapture = new AudioCapture(16000); // 16kHz

        Console.WriteLine("开始录音，按 Enter 停止...");

        var stream = recognizer.CreateStream();
        var tokenSource = new CancellationTokenSource();

        // 启动音频处理任务
        var audioTask = Task.Run(async () =>
        {
            while (!tokenSource.Token.IsCancellationRequested)
            {
                var samples = audioCapture.Read(1600); // 100ms
                if (samples.Length > 0)
                {
                    stream.AcceptWaveform(samples, 16000);
                    recognizer.Decode(stream);

                    var result = recognizer.GetResult(stream);
                    if (!string.IsNullOrEmpty(result.Text))
                    {
                        Console.WriteLine($"\r{result.Text}");
                    }
                }
                await Task.Delay(100);
            }
        }, tokenSource.Token);

        // 等待用户输入
        Console.ReadLine();
        tokenSource.Cancel();

        audioTask.Wait();
    }
}
```

### 语音合成

```csharp
using SherpaOnnx;
using System;
using System.IO;

class TextToSpeech
{
    static void Main(string[] args)
    {
        // 配置 TTS 模型
        var modelConfig = new OfflineTtsModelConfig()
        {
            Vits = new OfflineTtsVitsModelConfig()
            {
                Model = "./vits.onnx",
                Lexicon = "./lexicon.txt",
                Tokens = "./tokens.txt"
            },
            NumThreads = 1
        };

        // 创建 TTS 模型
        using var model = new OfflineTtsModel(modelConfig);

        // 生成语音
        var text = "Hello, world! 你好，世界！";
        var audio = model.Generate(text, 0); // 说话人 ID = 0

        // 保存音频
        var outputFile = "output.wav";
        File.WriteAllBytes(outputFile, audio.WavData);

        Console.WriteLine($"语音已保存到: {outputFile}");
        Console.WriteLine($"采样率: {audio.SampleRate} Hz");
        Console.WriteLine($"帧数: {audio.NumFrames}");

        // 播放音频 (Windows)
        if (Environment.OSVersion.Platform == PlatformID.Win32NT)
        {
            new SoundPlayer(outputFile).Play();
        }
    }
}
```

### 关键词检测

```csharp
using SherpaOnnx;
using System;
using System.Collections.Generic;
using System.IO;

class KeywordSpotting
{
    static void Main(string[] args)
    {
        // 创建关键词文件
        var keywords = new Dictionary<string, float>()
        {
            ["hello world"] = 0.5f,
            ["你好"] = 0.5f,
            ["语音识别"] = 0.3f
        };

        var keywordsFile = "keywords.txt";
        using (var writer = new StreamWriter(keywordsFile))
        {
            foreach (var kw in keywords)
            {
                writer.WriteLine($"{kw.Key} {kw.Value}");
            }
        }

        // 配置关键词检测器
        var modelConfig = new KeywordSpotterModelConfig()
        {
            Transducer = new OnlineTransducerModelConfig()
            {
                Encoder = "./encoder.onnx",
                Decoder = "./decoder.onnx",
                Joiner = "./joiner.onnx"
            },
            Tokens = "./tokens.txt"
        };

        var config = new KeywordSpotterConfig()
        {
            ModelConfig = modelConfig,
            KeywordsFile = keywordsFile,
            KeywordsThreshold = 0.3f
        };

        using var spotter = new KeywordSpotter(config);
        using var stream = spotter.CreateStream();

        // 处理音频
        var audioFile = "./test.wav";
        var waveData = WaveReader.Read(audioFile);

        stream.AcceptWaveform(waveData.Samples, waveData.SampleRate);
        spotter.Decode(stream);

        var result = spotter.GetResult(stream);
        if (!string.IsNullOrEmpty(result.Keyword))
        {
            Console.WriteLine($"检测到关键词: {result.Keyword}");
            Console.WriteLine($"置信度: {result.Prob:F2}");
        }
    }
}
```

### 说话人分离

```csharp
using SherpaOnnx;
using System;
using System.Collections.Generic;

class SpeakerDiarization
{
    static void Main(string[] args)
    {
        // 配置说话人分离
        var segmentationConfig = new SegmentorModelConfig()
        {
            SherpaDiarization = new SherpaDiarizationConfig()
            {
                Model = "./seg_model.onnx"
            },
            NumThreads = 2
        };

        var embeddingConfig = new SpeakerEmbeddingExtractorConfig()
        {
            SpeakerEmbedding = new SpeakerEmbeddingConfig()
            {
                Model = "./embedding_model.onnx"
            },
            NumThreads = 2
        };

        // 创建模型
        using var segmenter = new SegmentorModel(segmentationConfig);
        using var extractor = new SpeakerEmbeddingExtractor(embeddingConfig);

        var diarizationConfig = new SpeakerDiarizationConfig()
        {
            Segmentor = segmenter,
            EmbeddingExtractor = extractor,
            Clustering = new ClusteringConfig()
            {
                NumClusters = 2,
                Threshold = 0.5f
            }
        };

        using var diarization = new SpeakerDiarization(diarizationConfig);

        // 处理音频
        var audioFile = "./meeting.wav";
        var waveData = WaveReader.Read(audioFile);

        var segments = diarization.Process(
            waveData.Samples,
            waveData.SampleRate
        );

        // 输出结果
        Console.WriteLine("说话人分离结果:");
        for (int i = 0; i < segments.Count; i++)
        {
            var seg = segments[i];
            Console.WriteLine(
                $"段 {i + 1}: 说话人 {seg.Speaker} " +
                $"({seg.Start:F2}s - {seg.End:F2}s)"
            );
        }
    }
}
```

## 项目结构

### Common 共享代码
包含共享的工具类和扩展：
- **WaveReader**: WAV 文件读取
- **AudioCapture**: 音频捕获
- **ConsoleHelper**: 控制台辅助

### 各示例项目结构
```
example-name/
├── example-name.csproj    # 项目文件
├── Program.cs            # 主程序
├── README.md             # 说明文档
└── run.sh/.bat           # 运行脚本
```

## 构建和运行

### 1. 运行单个示例
```bash
cd offline-decode-files
dotnet run
```

### 2. 构建发布版本
```bash
dotnet publish -c Release -r win-x64 --self-contained
dotnet publish -c Release -r linux-x64 --self-contained
dotnet publish -c Release -r osx-x64 --self-contained
```

### 3. 使用 Docker
```dockerfile
FROM mcr.microsoft.com/dotnet/runtime:6.0 AS base
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["src/SherpaOnnxExample.csproj", "SherpaOnnxExample/"]
RUN dotnet restore "SherpaOnnxExample/SherpaOnnxExample.csproj"
COPY . .
WORKDIR "/src/SherpaOnnxExample"
RUN dotnet build "SherpaOnnxExample.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "SherpaOnnxExample.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "SherpaOnnxExample.dll"]
```

## 高级用法

### 1. 异步处理
```csharp
using System.Threading.Tasks;

public async Task<string> RecognizeAsync(string audioFile)
{
    return await Task.Run(() =>
    {
        using var recognizer = new OfflineRecognizer(modelConfig);
        // ... 识别逻辑 ...
        return result.Text;
    });
}
```

### 2. 流式处理
```csharp
public IEnumerable<string> RecognizeStream(Stream audioStream)
{
    using var reader = new BinaryReader(audioStream);
    var buffer = new byte[3200]; // 100ms @ 16kHz

    while (reader.Read(buffer, 0, buffer.Length) > 0)
    {
        var samples = ConvertToSamples(buffer);
        // 处理音频块...
        yield return result.Text;
    }
}
```

### 3. 多语言支持
```csharp
var languageConfig = new Dictionary<string, OfflineModelConfig>()
{
    ["en"] = GetEnglishModelConfig(),
    ["zh"] = GetChineseModelConfig(),
    ["ja"] = GetJapaneseModelConfig()
};

var recognizer = new OfflineRecognizer(languageConfig[targetLanguage]);
```

## 性能优化

### 1. 使用对象池
```csharp
using Microsoft.Extensions.ObjectPool;

public class RecognizerPool
{
    private readonly ObjectPool<OfflineRecognizer> _pool;

    public RecognizerPool()
    {
        _pool = new DefaultObjectPoolProvider()
            .Create(new RecognizerPooledObjectPolicy());
    }

    public string Recognize(byte[] audio)
    {
        var recognizer = _pool.Get();
        try
        {
            // 使用识别器
            return result.Text;
        }
        finally
        {
            _pool.Return(recognizer);
        }
    }
}
```

### 2. 并行处理
```csharp
using System.Threading.Tasks.Dataflow;

var block = new ActionBlock<string>(async file =>
{
    var result = await RecognizeAsync(file);
    Console.WriteLine($"{file}: {result}");
},
new ExecutionDataflowBlockOptions
{
    MaxDegreeOfParallelism = Environment.ProcessorCount
});

// 添加文件到处理队列
foreach (var file in audioFiles)
{
    block.Post(file);
}

block.Complete();
await block.Completion;
```

## 故障排除

### 1. DLL 加载失败
确保 ONNX Runtime 库在系统路径中：
```csharp
// 手动设置库路径
Environment.SetEnvironmentVariable(
    "PATH",
    Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "runtimes") +
    Path.PathSeparator +
    Environment.GetEnvironmentVariable("PATH")
);
```

### 2. 模型路径错误
使用绝对路径或确保相对路径正确：
```csharp
var modelPath = Path.Combine(
    AppDomain.CurrentDomain.BaseDirectory,
    "models",
    "encoder.onnx"
);
```

### 3. 内存泄漏
使用 `using` 语句确保资源释放：
```csharp
using var recognizer = new OfflineRecognizer(modelConfig);
using var stream = recognizer.CreateStream();
```

## 相关文档

- [.NET 官方文档](https://docs.microsoft.com/dotnet/)
- [Sherpa-ONNX C# API](https://k2-fsa.github.io/sherpa/onnx/csharp-api/)
- [NuGet 包](https://www.nuget.org/packages/SherpaOnnx/)
- [.NET 示例项目](https://github.com/k2-fsa/sherpa-onnx/tree/master/dotnet-examples)

---

*更多示例和详细信息请参考项目中的各个示例目录。*