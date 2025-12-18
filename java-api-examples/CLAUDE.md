[根目录](../../CLAUDE.md) > **java-api-examples**

# Sherpa-ONNX Java API 示例

> 更新时间：2025-12-18 10:45:53

## 模块职责

本模块提供了 Sherpa-ONNX Java API 的完整使用示例，涵盖了所有主要功能的使用方法，包括语音识别（ASR）、语音合成（TTS）、说话人识别、语音增强等。每个示例都是一个独立的 Java 程序，展示了如何配置和使用相应的功能。

## 入口与启动

### 运行示例
```bash
# 编译示例
javac -cp ".:path/to/sherpa-onnx-java-api.jar" *.java

# 运行示例
java -cp ".:path/to/sherpa-onnx-java-api.jar" -Djava.library.path=path/to/native ExampleName
```

### 使用脚本运行
每个示例都配有对应的 shell 脚本：
```bash
# 运行 Paraformer 语音识别示例
./run-non-streaming-decode-file-paraformer.sh

# 运行 VITS 语音合成示例
./run-non-streaming-tts-vits-zh.sh
```

## 主要示例分类

### 1. 语音识别 (ASR)

#### 非流式识别
- `NonStreamingDecodeFileParaformer.java` - Paraformer 模型
- `NonStreamingDecodeFileTransducer.java` - Transducer 模型
- `NonStreamingDecodeFileWhisper.java` - Whisper 模型
- `NonStreamingDecodeFileSenseVoice.java` - SenseVoice 模型
- `NonStreamingDecodeFileMoonshine.java` - Moonshine 模型
- `NonStreamingDecodeFileDolphinCtc.java` - Dolphin CTC 模型
- `NonStreamingDecodeFileFireRedAsr.java` - FireRed ASR 模型
- `NonStreamingDecodeFileNemo.java` - NeMo CTC 模型
- `NonStreamingDecodeFileOmnilingualAsrCtc.java` - Omnilingual 模型
- `NonStreamingDecodeFileZipformerCtc.java` - Zipformer CTC 模型

#### 流式识别
- `StreamingDecodeFileParaformer.java` - Paraformer 流式识别
- `StreamingDecodeFileTransducer.java` - Transducer 流式识别
- `StreamingDecodeFileCtc.java` - CTC 流式识别
- `StreamingDecodeFileToneCtc.java` - Tone CTC 流式识别
- `StreamingDecodeFileCtcHLG.java` - HLG 解码
- `StreamingAsrFromMicTransducer.java` - 麦克风实时识别

### 2. 语音合成 (TTS)
- `NonStreamingTtsVitsZh.java` - VITS 中文 TTS
- `NonStreamingTtsMatchaEn.java` - Matcha 英文 TTS
- `NonStreamingTtsMatchaZh.java` - Matcha 中文 TTS
- `NonStreamingTtsKokoroEn.java` - Kokoro 英文 TTS
- `NonStreamingTtsKokoroZhEn.java` - Kokoro 中英混合 TTS
- `NonStreamingTtsKittenEn.java` - Kitten 英文 TTS
- `NonStreamingTtsPiperEn.java` - Piper 英文 TTS
- `NonStreamingTtsCoquiDe.java` - Coqui 德文 TTS
- `NonStreamingTtsPiperEnWithCallback.java` - 带回调的 TTS

### 3. 说话人处理
- `SpeakerIdentification.java` - 说话人识别
- `OfflineSpeakerDiarizationDemo.java` - 说话人日志
- `OfflineAddPunctuation.java` - 离线标点恢复
- `OnlineAddPunctuation.java` - 在线标点恢复

### 4. 其他功能
- `VadFromMic.java` - 麦克风 VAD
- `VadRemoveSilence.java` - VAD 静音移除
- `TenVadRemoveSilence.java` - Ten VAD
- `AudioTaggingCED.java` - CED 音频标注
- `AudioTaggingZipformer.java` - Zipformer 音频标注
- `KeywordSpotterFromFile.java` - 关键词检测
- `SpokenLanguageIdentificationWhisper.java` - 语言识别
- `NonStreamingSpeechEnhancementGtcrn.java` - 语音增强

### 5. 高级功能
- `InverseTextNormalizationNonStreamingParaformer.java` - 反向文本标准化
- `InverseTextNormalizationStreamingTransducer.java` - 流式反向文本标准化
- `NonStreamingWebsocketClient.java` - WebSocket 客户端
- `VersionTest.java` - 版本测试

## 使用示例

### 基础语音识别
```java
import com.k2fsa.sherpa.onnx.*;

public class BasicASR {
    public static void main(String[] args) {
        // 配置模型
        OfflineParaformerModelConfig paraformer = new OfflineParaformerModelConfig();
        paraformer.model = "./models/paraformer/model.onnx";

        OfflineModelConfig modelConfig = new OfflineModelConfig();
        modelConfig.paraformer = paraformer;
        modelConfig.tokens = "./models/tokens.txt";
        modelConfig.numThreads = 2;

        // 创建识别器
        OfflineRecognizerConfig config = new OfflineRecognizerConfig();
        config.modelConfig = modelConfig;
        config.decodingMethod = "greedy_search";

        OfflineRecognizer recognizer = new OfflineRecognizer(config);

        // 识别音频文件
        OfflineStream stream = recognizer.createStream();
        WaveReader reader = new WaveReader("test.wav");
        stream.acceptWaveform(reader.getSamples(), reader.getSampleRate());

        // 获取结果
        recognizer.decode(stream);
        OfflineRecognizerResult result = recognizer.getResult(stream);
        System.out.println("识别结果: " + result.text);
    }
}
```

### 语音合成
```java
import com.k2fsa.sherpa.onnx.*;

public class BasicTTS {
    public static void main(String[] args) {
        // 配置模型
        OfflineTtsVitsModelConfig vits = new OfflineTtsVitsModelConfig();
        vits.model = "./models/vits/model.onnx";
        vits.lexicon = "./models/lexicon.txt";
        vits.tokens = "./models/tokens.txt";

        OfflineTtsModelConfig modelConfig = new OfflineTtsModelConfig();
        modelConfig.vits = vits;
        modelConfig.numThreads = 1;

        // 创建 TTS
        OfflineTtsConfig config = new OfflineTtsConfig();
        config.modelConfig = modelConfig;
        config.ruleFsts = "";
        config.maxNumSentences = 2;

        OfflineTts tts = new OfflineTts(config);

        // 生成语音
        String text = "你好，世界！";
        GeneratedAudio audio = tts.generate(text);

        // 保存音频
        WaveWriter.write("output.wav", audio.samples, audio.sampleRate);
        System.out.println("生成成功！采样率: " + audio.sampleRate);
    }
}
```

### 实时麦克风识别
```java
import com.k2fsa.sherpa.onnx.*;

public class MicrophoneASR {
    public static void main(String[] args) {
        // 配置流式识别器
        OnlineTransducerModelConfig transducer = new OnlineTransducerModelConfig();
        transducer.encoder = "./models/encoder.onnx";
        transducer.decoder = "./models/decoder.onnx";
        transducer.joiner = "./models/joiner.onnx";

        OnlineModelConfig modelConfig = new OnlineModelConfig();
        modelConfig.transducer = transducer;
        modelConfig.tokens = "./models/tokens.txt";

        OnlineRecognizerConfig config = new OnlineRecognizerConfig();
        config.modelConfig = modelConfig;
        config.enableEndpoint = true;

        OnlineRecognizer recognizer = new OnlineRecognizer(config);
        OnlineStream stream = recognizer.createStream();

        // 配置音频采集（需要额外的音频库）
        // 这里仅展示识别部分
        // ...

        // 实时处理
        while (recording) {
            // 获取音频数据
            float[] samples = getAudioSamples();
            stream.acceptWaveform(samples);

            // 解码
            recognizer.decode(stream);

            // 获取结果
            if (recognizer.isEndpoint(stream)) {
                OnlineRecognizerResult result = recognizer.getResult(stream);
                System.out.println("结果: " + result.text);
                recognizer.reset(stream);
            }
        }
    }
}
```

## 模型配置

### 下载模型
示例脚本会自动下载模型，也可以手动下载：
```bash
# Paraformer 中文模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-paraformer-zh-2023-03-28.tar.bz2

# Whisper 英文模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-whisper-base.tar.bz2
```

### 配置文件位置
- 模型文件通常放在 `./models/` 目录下
- 脚本中已预设了模型路径
- 可以根据需要修改模型路径

## 测试与质量

### 运行所有示例
```bash
# 批量运行 ASR 示例
for script in run-non-streaming-decode-file-*.sh; do
    echo "运行 $script"
    bash $script
done

# 批量运行 TTS 示例
for script in run-non-streaming-tts-*.sh; do
    echo "运行 $script"
    bash $script
done
```

### 常见问题
1. **JNI 库加载失败**
   - 确保 `libsherpa-onnx-jni` 在系统路径中
   - 设置 `-Djava.library.path` 参数

2. **模型文件未找到**
   - 检查模型路径是否正确
   - 运行脚本确保模型已下载

3. **内存不足**
   - 增加 JVM 堆内存：`-Xmx4g`
   - 使用较小的模型

## 相关文件清单

### 示例代码
- `*.java` - 所有 Java 示例程序
- `run-*.sh` - 运行脚本
- `.gitignore` - Git 忽略文件

### 文档
- `README.md` - 使用说明

## 变更记录 (Changelog)

### 2025-12-18 10:45:53
- ✨ 创建 Java API 示例模块文档
- 📊 初始文档覆盖率 0%
- 📝 分类整理所有示例代码
- 💡 添加使用示例和配置说明

---

*更多信息和问题反馈请访问：https://github.com/k2-fsa/sherpa-onnx/tree/master/java-api-examples*