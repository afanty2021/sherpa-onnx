[根目录](../CLAUDE.md) > **ios**

# iOS 集成

> 更新时间：2025-12-10 07:48:28

## 模块职责

Sherpa-ONNX 的 iOS 集成模块提供了完整的 iOS 平台语音处理解决方案。它通过 Swift/Objective-C 绑定将 C++ 核心功能封装为 iOS 原生 API，支持以下功能：

- **实时语音识别**：流式 ASR，支持麦克风输入
- **语音合成**：TTS 功能，支持多种声音
- **语言识别**：识别语音语言
- **字幕生成**：音频/视频字幕实时生成
- **两遍识别**：快速初识别 + 精确调整

## 项目结构

### iOS 示例项目

```
ios-swiftui/                    # SwiftUI 示例应用
├── SherpaOnnx/                # 基础语音识别应用
├── SherpaOnnx2Pass/           # 两遍识别应用
├── SherpaOnnxTts/             # TTS 语音合成应用
├── SherpaOnnxLangID/          # 语言识别应用
└── SherpaOnnxSubtitle/        # 字幕生成应用

ios-swift/                     # UIKit 示例应用
└── SherpaOnnx/                # UIKit 版本示例
```

### 核心组件

```
每个示例项目包含：
├── Model.swift                # 模型配置
├── ViewModel.swift            # 业务逻辑
├── ContentView.swift          # SwiftUI 界面
├── ViewController.swift       # UIKit 界面
└── Extension.swift            # 扩展工具
```

## 入口与启动

### 1. 环境要求

- **Xcode**：14.0+
- **iOS 部署目标**：13.0+
- **Swift**：5.5+
- **设备**：支持 iOS 的 iPhone/iPad

### 2. 快速集成

#### 步骤 1：添加 Sherpa-ONNX 依赖

```swift
// 使用 Swift Package Manager
// File > Add Packages...
// 输入：https://github.com/k2-fsa/sherpa-onnx.git
```

#### 步骤 2：配置模型文件

1. 下载预训练模型：
```bash
# 中文+英文双语模型
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-paraformer-bilingual-zh-en-2023-02-20.tar.bz2
tar xf sherpa-onnx-streaming-paraformer-bilingual-zh-en-2023-02-20.tar.bz2
```

2. 添加到 Xcode 项目：
```
Build Phases > Copy Bundle Resources > 添加模型文件
```

#### 步骤 3：基础代码示例

```swift
import SwiftUI
import SherpaOnnx

struct ContentView: View {
    @StateObject private var viewModel = SherpaOnnxViewModel()

    var body: some View {
        VStack(spacing: 20) {
            Text("实时语音识别")
                .font(.largeTitle)

            Text(viewModel.subtitles)
                .padding()
                .background(Color.gray.opacity(0.1))
                .cornerRadius(10)

            Button(action: {
                if viewModel.status == .stop {
                    viewModel.startRecording()
                } else {
                    viewModel.stopRecording()
                }
            }) {
                Text(viewModel.status == .stop ? "开始" : "停止")
                    .padding()
                    .background(viewModel.status == .stop ? Color.green : Color.red)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
        }
        .padding()
    }
}
```

## 对外接口

### 1. 语音识别接口

#### 配置模型
```swift
// 在 Model.swift 中配置
func getBilingualStreamingZhEnParaformer() -> SherpaOnnxOnlineModelConfig {
    let encoder = getResource("encoder.int8", "onnx")
    let decoder = getResource("decoder.int8", "onnx")
    let tokens = getResource("tokens", "txt")

    return sherpaOnnxOnlineModelConfig(
        tokens: tokens,
        paraformer: sherpaOnnxOnlineParaformerModelConfig(
            encoder: encoder,
            decoder: decoder
        ),
        numThreads: 1,
        modelType: "paraformer"
    )
}
```

#### 识别器初始化
```swift
class SherpaOnnxViewModel: ObservableObject {
    private var recognizer: SherpaOnnxRecognizer!

    private func initRecognizer() {
        let modelConfig = getBilingualStreamingZhEnParaformer()

        let featConfig = sherpaOnnxFeatureConfig(
            sampleRate: 16000,
            featureDim: 80
        )

        var config = sherpaOnnxOnlineRecognizerConfig(
            featConfig: featConfig,
            modelConfig: modelConfig,
            enableEndpoint: true,
            rule1MinTrailingSilence: 2.4,
            rule2MinTrailingSilence: 0.8,
            rule3MinUtteranceLength: 30,
            decodingMethod: "greedy_search",
            maxActivePaths: 4
        )

        recognizer = SherpaOnnxRecognizer(config: &config)
    }
}
```

#### 实时识别
```swift
private func initRecorder() {
    audioEngine = AVAudioEngine()

    // 获取输入节点
    let inputNode = audioEngine!.inputNode
    let inputFormat = inputNode.outputFormat(forBus: 0)

    // 设置音频处理格式
    let recordingFormat = AVAudioFormat(
        commonFormat: .pcmFormatFloat32,
        sampleRate: 16000,
        channels: 1,
        interleaved: false
    )!

    // 安装音频处理回调
    inputNode.installTap(
        onBus: 0,
        bufferSize: 4096,
        format: recordingFormat
    ) { [weak self] buffer, _ in
            guard let self = self,
                  let channelData = buffer.floatChannelData?[0] else {
                return
            }

            // 处理音频数据
            let array = Array(UnsafeBufferPointer(
                start: channelData,
                count: Int(buffer.frameLength)
            ))

            // 识别
            self.recognizer.acceptWaveform(samples: array)

            // 获取结果
            if self.recognizer.isReady() {
                self.recognizer.decode()
                let result = self.recognizer.getResult()
                DispatchQueue.main.async {
                    self.updateResult(result.text)
                }
            }
        }
}
```

### 2. 语音合成接口

```swift
class TtsViewModel: ObservableObject {
    private var tts: SherpaOnnxOfflineTts!

    private func initTts() {
        let modelConfig = SherpaOnnxOfflineTtsModelConfig(
            vits: SherpaOnnxOfflineTtsVitsModelConfig(
                model: getResource("model", "onnx"),
                lexicon: getResource("lexicon", "txt"),
                tokens: getResource("tokens", "txt")
            ),
            numThreads: 1,
            debug: true,
            provider: "cpu"
        )

        let config = SherpaOnnxOfflineTtsConfig(
            model: modelConfig,
            ruleFsts: "",
            ruleFars: "",
            maxNumSentences: 2
        )

        tts = SherpaOnnxOfflineTts(config: config)
    }

    func generateSpeech(text: String) {
        let audio = tts.generate(text: text, speakerId: 0, speed: 1.0)

        // 播放音频
        let avAudioBuffer = AVAudioPCMBuffer(
            pcmFormat: AVAudioFormat(
                commonFormat: .pcmFormatFloat32,
                sampleRate: Double(audio.sampleRate),
                channels: 1,
                interleaved: false
            )!,
            frameCapacity: AVAudioFrameCount(audio.samples.count)
        )!

        avAudioBuffer.floatChannelData![0].assign(
            from: audio.samples,
            count: audio.samples.count
        )

        let playerNode = AVAudioPlayerNode()
        audioEngine.attach(playerNode)
        audioEngine.connect(playerNode, to: audioEngine.mainMixerNode, format: nil)

        playerNode.scheduleBuffer(avAudioBuffer)
        audioEngine.start()
        playerNode.play()
    }
}
```

### 3. 字幕生成接口

```swift
class SubtitleViewModel: ObservableObject {
    @Published var subtitles: [SpeechSegment] = []

    func generateSubtitles(from url: URL) async {
        // 加载音频文件
        let asset = AVAsset(url: url)
        guard let track = asset.tracks(withMediaType: .audio).first else { return }

        // 创建识别器
        let recognizer = SherpaOnnxRecognizer(config: &config)

        // 读取音频数据
        let reader = try! AVAssetReader(asset)
        let output = AVAssetReaderTrackOutput(
            track: track,
            outputSettings: [
                AVFormatIDKey: kAudioFormatLinearPCM,
                AVLinearPCMIsFloatKey: true,
                AVLinearPCMIsBigEndianKey: false,
                AVLinearPCMBitDepthKey: 32,
                AVLinearPCMIsNonInterleaved: true
            ]
        )

        reader.add(output)
        reader.startReading()

        // 分段处理
        var segmentCount = 0
        let segmentDuration: Double = 10.0  // 10秒一段

        while reader.status == .reading {
            guard let buffer = output.copyNextSampleBuffer() else { continue }

            // 处理音频片段
            let audioData = extractAudioData(from: buffer)
            recognizer.acceptWaveform(samples: audioData)

            if recognizer.isReady() {
                recognizer.decode()
                let result = recognizer.getResult()

                if !result.text.isEmpty {
                    let startTime = Double(segmentCount) * segmentDuration
                    let segment = SpeechSegment(
                        startTime: startTime,
                        endTime: startTime + segmentDuration,
                        text: result.text
                    )

                    await MainActor.run {
                        subtitles.append(segment)
                    }
                }
            }

            segmentCount += 1
        }
    }
}
```

## 关键依赖与配置

### 系统框架
- **AVFoundation**：音频采集和播放
- **Combine**：响应式编程
- **SwiftUI**：声明式 UI
- **Foundation**：基础功能

### 权限配置

在 `Info.plist` 中添加：
```xml
<key>NSMicrophoneUsageDescription</key>
<string>此应用需要访问麦克风进行语音识别</string>

<key>NSDocumentsFolderUsageDescription</key>
<string>此应用需要访问文档文件夹加载音频文件</string>
```

### 构建设置

```swift
// Package.swift
// swift-tools-version:5.7
import PackageDescription

let package = Package(
    name: "SherpaOnnxExample",
    platforms: [
        .iOS(.v13)
    ],
    products: [
        .library(name: "SherpaOnnx", targets: ["SherpaOnnx"])
    ],
    dependencies: [
        .package(url: "https://github.com/k2-fsa/sherpa-onnx.git", from: "1.12.0")
    ],
    targets: [
        .target(
            name: "SherpaOnnx",
            dependencies: ["SherpaOnnx"],
            path: "Sources",
            resources: [
                .process("Models")
            ]
        )
    ]
)
```

## 性能优化

### 1. 音频处理优化

```swift
// 使用合适的缓冲区大小
let optimalBufferSize: AVAudioFrameCount = 4096

// 降低延迟
inputNode.installTap(
    onBus: 0,
    bufferSize: optimalBufferSize,
    format: recordingFormat
) { buffer, _ in
    // 处理音频
}

// 后台处理识别
DispatchQueue.global(qos: .userInteractive).async {
    self.recognizer.decode()
}
```

### 2. 内存管理

```swift
class SpeechRecognizer {
    private var recognizer: SherpaOnnxRecognizer?

    deinit {
        // 清理资源
        recognizer = nil
        audioEngine?.stop()
        audioEngine = nil
    }

    func reset() {
        // 重置识别器状态
        recognizer?.reset()
    }
}
```

### 3. 模型优化

```swift
// 使用量化模型
func getQuantizedModelConfig() -> SherpaOnnxOnlineModelConfig {
    // 使用 .int8 模型
    let encoder = getResource("encoder.int8", "onnx")
    let decoder = getResource("decoder.int8", "onnx")

    // 配置
    return sherpaOnnxOnlineModelConfig(
        tokens: tokens,
        paraformer: sherpaOnnxOnlineParaformerModelConfig(
            encoder: encoder,
            decoder: decoder
        ),
        numThreads: 1,  // 减少线程数
        modelType: "paraformer"
    )
}
```

## 部署和发布

### 1. App Store 配置

```swift
// 在 Xcode 中设置：
// - Deployment Target: iOS 13.0+
// - Architectures: arm64
// - Valid Architectures: arm64
// - Build Active Architecture Only: Debug = YES, Release = NO
```

### 2. 模型文件处理

```swift
// 使用 Asset Catalog 或 Bundle Resources
// 避免模型文件被压缩
// 启用 "On Demand Resources" 减小应用体积

// 动态下载模型
func downloadModelIfNeeded() async {
    let modelURL = URL(string: "https://example.com/model.onnx")!
    let destination = getDocumentsDirectory().appendingPathComponent("model.onnx")

    // 检查本地是否存在
    if FileManager.default.fileExists(atPath: destination.path) {
        return
    }

    // 下载模型
    let (data, _) = try await URLSession.shared.data(from: modelURL)
    try data.write(to: destination)
}
```

### 3. 性能监控

```swift
import os.log

class PerformanceMonitor {
    private let logger = Logger(subsystem: "com.app.sherpaonnx", category: "Performance")

    func measureRecognitionTime<T>(operation: () -> T) -> T {
        let startTime = CFAbsoluteTimeGetCurrent()
        let result = operation()
        let timeElapsed = CFAbsoluteTimeGetCurrent() - startTime

        logger.info("Recognition took \(timeElapsed * 1000, privacy: .public) ms")

        return result
    }
}
```

## 测试策略

### 1. 单元测试

```swift
import XCTest
@testable import SherpaOnnx

class SherpaOnnxTests: XCTestCase {
    func testModelLoading() {
        let config = getTestModelConfig()
        let recognizer = SherpaOnnxRecognizer(config: &config)
        XCTAssertNotNil(recognizer)
    }

    func testAudioProcessing() {
        let testData = generateTestAudio()
        let result = processAudio(testData)
        XCTAssertFalse(result.isEmpty)
    }
}
```

### 2. UI 测试

```swift
class SherpaOnnxUITests: XCTestCase {
    func testVoiceRecognitionFlow() throws {
        let app = XCUIApplication()
        app.launch()

        // 点击开始按钮
        app.buttons["开始"].tap()

        // 等待识别结果
        let resultText = app.textViews["results"]
        let exists = resultText.waitForExistence(timeout: 10)
        XCTAssertTrue(exists)
    }
}
```

## 常见问题 (FAQ)

### Q1: 识别延迟过高？
A: 解决方案：
- 减小音频缓冲区大小
- 使用更小的模型
- 启用 VAD 端点检测

### Q2: 内存占用过大？
A: 优化方法：
```swift
// 使用弱引用避免循环引用
audioEngine?.inputNode.installTap { [weak self] buffer, _ in
    // 处理音频
}

// 及时释放资源
deinit {
    audioEngine?.stop()
    audioEngine?.inputNode.removeTap(onBus: 0)
}
```

### Q3: 模型文件太大？
A: 解决方案：
- 使用量化模型
- 实现动态下载
- 压缩模型文件

### Q4: 实时识别中断？
A: 检查：
- 音频会话配置
- 后台权限
- 内存使用情况

### Q5: 麦克风权限被拒绝？
A: 处理方式：
```swift
func checkMicrophonePermission() {
    AVAudioSession.sharedInstance().requestRecordPermission { granted in
        if !granted {
            // 显示权限说明
            DispatchQueue.main.async {
                let alert = UIAlertController(
                    title: "需要麦克风权限",
                    message: "请在设置中允许访问麦克风",
                    preferredStyle: .alert
                )

                alert.addAction(UIAlertAction(
                    title: "去设置",
                    style: .default
                ) { _ in
                    if let settings = URL(string: UIApplication.openSettingsURLString) {
                        UIApplication.shared.open(settings)
                    }
                })

                self.present(alert, animated: true)
            }
        }
    }
}
```

## 相关文件清单

### 示例应用
- `ios-swiftui/SherpaOnnx/` - 基础语音识别
- `ios-swiftui/SherpaOnnx2Pass/` - 两遍识别
- `ios-swiftui/SherpaOnnxTts/` - 语音合成
- `ios-swiftui/SherpaOnnxSubtitle/` - 字幕生成
- `ios-swiftui/SherpaOnnxLangID/` - 语言识别

### 核心文件
- `Model.swift` - 模型配置
- `ViewModel.swift` - 业务逻辑
- `ContentView.swift` - SwiftUI 界面
- `ViewController.swift` - UIKit 界面

### 构建文件
- `Package.swift` - Swift Package 配置
- `.xcodeproj/` - Xcode 项目文件
- `Info.plist` - 应用配置

## 变更记录

### 2025-12-10 07:48:28
- 📝 创建 iOS 集成文档
- 📱 添加 SwiftUI 和 UIKit 示例
- 🎯 包含完整的 API 使用说明
- 🔧 补充性能优化和故障排除

---

*相关文件：[Swift 示例](../swift-api-examples/CLAUDE.md) | [构建系统](../build.sh) | [模型下载](https://github.com/k2-fsa/sherpa-onnx/releases)*