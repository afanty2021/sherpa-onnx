[根目录](../CLAUDE.md) > **android**

# Sherpa-ONNX Android 支持

> 更新时间：2025-12-10 07:44:45

## 模块职责

Android 模块提供了 Sherpa-ONNX 在 Android 平台的完整解决方案，包括：

1. **原生库集成**：通过 JNI 调用 C++ 核心功能
2. **应用示例**：展示各种语音处理功能的 Android 应用
3. **构建系统**：Gradle 配置和脚本自动化
4. **模型管理**：模型下载和部署策略

## 项目结构

```
android/
├── README.md                           # Android 使用说明
├── build-android-*.sh                 # 各架构构建脚本
├── SherpaOnnx/                        # 主应用（基础 ASR）
│   ├── app/
│   │   ├── src/main/java/             # Java/Kotlin 代码
│   │   ├── jniLibs/                   # 原生库
│   │   └── assets/                    # 模型文件
│   └── build.gradle
├── SherpaOnnx2Pass/                   # 两遍识别应用
├── SherpaOnnxAar/                     # Android Library (AAR)
├── SherpaOnnxTts/                     # TTS 应用
├── SherpaOnnxVad/                     # VAD 应用
├── SherpaOnnxVadAsr/                  # VAD + ASR 应用
├── SherpaOnnxTtsEngine/               # TTS 引擎服务
└── [其他功能应用]/
```

## 主要应用说明

### 1. SherpaOnnx（基础语音识别）
- **功能**：实时语音识别
- **模型**：支持多种流式/非流式模型
- **特性**：
  - 实时识别和显示
  - 支持多语言
  - 可配置模型参数
  - 音频可视化

### 2. SherpaOnnx2Pass（两遍识别）
- **功能**：快速初识别 + 精确再识别
- **优势**：低延迟 + 高精度
- **适用**：实时对话场景

### 3. SherpaOnnxTts（语音合成）
- **功能**：文本转语音
- **模型**：VITS、Piper、Matcha 等
- **特性**：
  - 多语种支持
  - 实时合成
  - 可调节语速

### 4. SherpaOnnxVad（语音活动检测）
- **功能**：检测语音活动
- **用途**：语音预处理、静音检测

### 5. 功能特化应用
- `SherpaOnnxAudioTagging` - 音频内容标记
- `SherpaOnnxSpeakerDiarization` - 说话人日志
- `SherpaOnnxSpeakerIdentification` - 说话人识别
- `SherpaOnnxSpokenLanguageIdentification` - 语言识别
- `SherpaOnnxKws` - 关键词检测

## 构建和部署

### 1. 环境准备
```bash
# 安装 Android NDK
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/25.1.8937393

# 安装 CMake
# Android Studio SDK Manager > SDK Tools > CMake
```

### 2. 构建 Android 库
```bash
# ARM64 (推荐)
./build-android-arm64-v8a.sh

# ARM32
./build-android-armv7-eabi.sh

# x86_64
./build-android-x86-64.sh
```

### 3. 集成到应用

#### 使用 AAR
```gradle
dependencies {
    implementation files('libs/sherpa-onnx.aar')
}
```

#### 使用原生库
```gradle
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['path/to/jniLibs']
        }
    }
}
```

### 4. 模型部署

#### Assets 目录结构
```
assets/
├── model/
│   ├── encoder-epoch-20-avg-1.onnx
│   ├── decoder-epoch-20-avg-1.onnx
│   └── joiner-epoch-20-avg-1.onnx
└── tokens.txt
```

#### 模型大小优化
- 使用量化模型（减少 4-8 倍）
- 模型剪枝
- 选择轻量级模型

## API 使用示例

### 1. Java API 基础使用
```java
import com.k2fsa.sherpa.onnx.OfflineRecognizer;
import com.k2fsa.sherpa.onnx.OfflineModelConfig;
import com.k2fsa.sherpa.onnx.OfflineRecognizerConfig;

// 1. 配置模型
OfflineModelConfig modelConfig = new OfflineModelConfig();
modelConfig.setZipformer("./model/zipformer.onnx");
modelConfig.setTokens("./model/tokens.txt");

OfflineRecognizerConfig config = new OfflineRecognizerConfig();
config.setModelConfig(modelConfig);
config.setNumThreads(2);

// 2. 创建识别器
OfflineRecognizer recognizer = new OfflineRecognizer(config);

// 3. 准备音频数据
float[] samples = readAudioFile();

// 4. 识别
String text = recognizer.recognize(samples);

// 5. 释放
recognizer.release();
```

### 2. 实时识别
```java
import com.k2fsa.sherpa.onnx.OnlineRecognizer;

// 创建在线识别器
OnlineRecognizer onlineRecognizer = new OnlineRecognizer(config);
OnlineStream stream = onlineRecognizer.createStream();

// 音频回调处理
AudioRecord audioRecord = new AudioRecord(...);
audioRecord.setRecordPositionUpdateListener(new OnRecordPositionUpdateListener() {
    @Override
    public void onPeriodicNotification(AudioRecord recorder) {
        float[] buffer = new float[1600];
        int samples = recorder.read(buffer, 0, buffer.length);

        // 提交给识别器
        stream.acceptWaveform(16000, buffer, samples);

        // 检查是否有结果
        if (onlineRecognizer.isReady(stream)) {
            onlineRecognizer.decode(stream);
            String text = onlineRecognizer.getResult(stream).getText();
            if (!text.isEmpty()) {
                updateUI(text);
            }
        }
    }
});
```

### 3. TTS 使用
```java
import com.k2fsa.sherpa.onnx.OfflineTts;
import com.k2fsa.sherpa.onnx.OfflineTtsModelConfig;

// 配置 TTS
OfflineTtsModelConfig ttsConfig = new OfflineTtsModelConfig();
ttsConfig.setVits(new VitsModelConfig("./model/vits.onnx"));
ttsConfig.setTokens("./model/tokens.txt");

// 创建 TTS
OfflineTts tts = new OfflineTts(ttsConfig);

// 生成语音
GeneratedAudio audio = tts.generate("Hello World", 1.0f, 0);

// 播放
playAudio(audio.getSamples(), audio.getSampleRate());
```

## 权限配置

在 AndroidManifest.xml 中添加必要权限：
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />  <!-- 如果需要下载模型 -->
```

动态权限请求（Android 6.0+）：
```java
ActivityCompat.requestPermissions(
    this,
    new String[]{Manifest.permission.RECORD_AUDIO},
    REQUEST_RECORD_AUDIO_PERMISSION
);
```

## 性能优化

### 1. 实时性能
```java
// 使用音频线程池
ExecutorService audioExecutor = Executors.newSingleThreadExecutor();

// 缓冲区优化
int bufferSize = AudioRecord.getMinBufferSize(
    SAMPLE_RATE,
    AudioFormat.CHANNEL_IN_MONO,
    AudioFormat.ENCODING_PCM_FLOAT
);
```

### 2. 内存优化
```java
// 复用识别器
private OfflineRecognizer recognizer;

// 及时释放资源
@Override
protected void onDestroy() {
    super.onDestroy();
    if (recognizer != null) {
        recognizer.release();
    }
}
```

### 3. 电池优化
- 使用适当采样率（16kHz 足够）
- 动态调整线程数
- 使用 Doze 模式兼容

## 常见问题

### Q: 如何减少 APK 大小？
A: 优化策略：
1. 使用 APK splits 按架构分包
2. 使用动态特性模块
3. 仅打包必要模型
4. 启用 R8 代码压缩

### Q: 如何处理音频焦点？
A: 实现音频焦点管理：
```java
AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
AudioFocusRequest focusRequest = new AudioFocusRequest.Builder(...)
    .setOnAudioFocusChangeListener(focusChangeListener)
    .build();

audioManager.requestAudioFocus(focusRequest);
```

### Q: 如何实现离线使用？
A: 完全离线方案：
1. 将模型打包到 assets
2. 首次运行时复制到内部存储
3. 禁用所有网络功能

## 发布和部署

### 1. 生成 APK
```bash
cd android/SherpaOnnx
./gradlew assembleDebug      # 调试版
./gradlew assembleRelease    # 发布版
```

### 2. 签名
```gradle
android {
    signingConfigs {
        release {
            storeFile file("keystore.jks")
            storePassword "password"
            keyAlias "key0"
            keyPassword "password"
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

### 3. ProGuard 配置
```proguard
-keep class com.k2fsa.sherpa.onnx.** { *; }
-dontwarn com.k2fsa.sherpa.onnx.**
```

## 变更记录

### 2025-12-10 07:44:45
- 📝 创建 Android 文档
- 📱 整理应用列表和功能
- 🔧 添加构建和使用指南
- ⚡ 记录性能优化建议

---

*相关文件：[Android 构建脚本](../build-android-*.sh) | [C++ 核心](../sherpa-onnx/csrc/CLAUDE.md) | [Java API](../sherpa-onnx/java/CLAUDE.md)*