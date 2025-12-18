# iOS Swift 集成

> 更新时间：2025-12-18 10:27:00

## 模块概述

本模块提供了 Sherpa-ONNX 在 iOS 平台上的 Swift 集成方案。它是一个完整的 iOS 应用示例，展示了如何在 Swift 应用中集成和使用 Sherpa-ONNX 进行实时语音识别。

## 项目结构

```
ios-swift/
└── SherpaOnnx/
    ├── SherpaOnnx/
    │   ├── AppDelegate.swift          # 应用委托
    │   ├── SceneDelegate.swift        # 场景委托
    │   ├── ViewController.swift       # 主视图控制器
    │   └── Model.swift               # 模型配置
    ├── SherpaOnnx.xcodeproj/         # Xcode 项目文件
    ├── SherpaOnnx.xcworkspace/       # Xcode 工作空间
    ├── SherpaOnnxTests/              # 单元测试
    └── SherpaOnnxUITests/            # UI 测试
```

## 核心组件

### ViewController.swift
主视图控制器，负责：
- 音频录制和播放管理
- 实时语音识别界面
- 识别结果展示
- 用户交互处理

主要功能：
- 使用 `AVAudioEngine` 进行音频采集
- 创建和管理 `SherpaOnnxRecognizer` 实例
- 处理音频缓冲区并传递给识别器
- 实时更新识别结果

### Model.swift
模型配置文件，预定义了多种模型配置：
- **getBilingualStreamZhEnZipformer20230220()**: 中英双语 Zipformer 模型
- **getZhZipformer20230615()**: 中文 Zipformer 模型
- **getZhZipformer20230615Int8()**: 中文量化模型
- **getEnZipformer20230626()**: 英文 Zipformer 模型
- **getBilingualStreamingZhEnParaformer()**: 中英双语 Paraformer 模型

## 快速开始

### 1. 环境要求
- Xcode 14.0+
- iOS 13.0+
- Swift 5.0+

### 2. 项目设置
```bash
# 克隆项目
git clone https://github.com/k2-fsa/sherpa-onnx.git
cd sherpa-onnx/ios-swift

# 打开项目
open SherpaOnnx/SherpaOnnx.xcworkspace
```

### 3. 添加模型文件
将下载的模型文件添加到 Xcode 项目的 `Build Phases -> Copy Bundle Resources`：
- encoder-*.onnx
- decoder-*.onnx
- joiner-*.onnx
- tokens.txt

### 4. 选择模型
在 `ViewController.swift` 中修改模型配置：
```swift
let modelConfig = getBilingualStreamZhEnZipformer20230220()
```

## 使用示例

### 基本语音识别

```swift
import AVFoundation
import SherpaOnnx

class ViewController: UIViewController {
    var recognizer: SherpaOnnxRecognizer!
    var audioEngine: AVAudioEngine?

    override func viewDidLoad() {
        super.viewDidLoad()

        // 配置模型
        let modelConfig = getBilingualStreamZhEnZipformer20230220()
        let recognizerConfig = sherpaOnnxOnlineRecognizerConfig(
            modelConfig: modelConfig,
            enableEndpoint: true,
            rule1MinSilenceDuration: 0.5,
            rule2MinSilenceDuration: 0.2
        )

        // 创建识别器
        recognizer = SherpaOnnxRecognizer(config: recognizerConfig)

        // 配置音频引擎
        setupAudioEngine()
    }

    func setupAudioEngine() {
        audioEngine = AVAudioEngine()
        let inputNode = audioEngine!.inputNode

        // 配置音频格式
        let recordingFormat = inputNode.outputFormat(forBus: 0)

        // 安装音频处理 tap
        inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { [weak self] buffer, _ in
            guard let self = self else { return }

            // 转换音频格式
            let audio = buffer.array()

            // 传递给识别器
            self.recognizer.acceptWaveform(samples: audio)

            // 执行识别
            self.recognizer.decode()

            // 获取结果
            if self.recognizer.isReady() {
                let result = self.recognizer.getResult()
                DispatchQueue.main.async {
                    // 更新 UI
                    self.updateUI(with: result.text)
                }
            }
        }
    }

    @IBAction func startRecording(_ sender: UIButton) {
        do {
            try audioEngine?.start()
            recognizer?.reset()
            sender.setTitle("停止", for: .normal)
        } catch {
            print("启动音频引擎失败: \(error)")
        }
    }
}
```

### 自定义模型配置

```swift
func createCustomModel() -> SherpaOnnxOnlineModelConfig {
    // 自定义模型文件路径
    let encoder = getResource("my-encoder", "onnx")
    let decoder = getResource("my-decoder", "onnx")
    let joiner = getResource("my-joiner", "onnx")
    let tokens = getResource("my-tokens", "txt")

    return sherpaOnnxOnlineModelConfig(
        tokens: tokens,
        transducer: sherpaOnnxOnlineTransducerModelConfig(
            encoder: encoder,
            decoder: decoder,
            joiner: joiner
        ),
        numThreads: 2,
        modelType: "zipformer2"
    )
}
```

## 高级功能

### 1. 实时因子 (RTF) 监控

```swift
func monitorPerformance() {
    let startTime = CACurrentMediaTime()

    // 执行识别...

    let endTime = CACurrentMediaTime()
    let processingTime = endTime - startTime
    let audioDuration = Double(audioSamples.count) / 16000.0

    let rtf = processingTime / audioDuration
    print("RTF: \(rtf)")
}
```

### 2. 热词支持

```swift
// 在识别器配置中添加热词
let recognizerConfig = sherpaOnnxOnlineRecognizerConfig(
    modelConfig: modelConfig,
    hotwordsFile: "hotwords.txt",
    hotwordsScore: 1.5
)
```

### 3. 端点检测

```swift
// 配置端点检测参数
let recognizerConfig = sherpaOnnxOnlineRecognizerConfig(
    modelConfig: modelConfig,
    enableEndpoint: true,
    rule1MinSilenceDuration: 2.0,  // 短停顿
    rule2MinSilenceDuration: 0.8,  // 长停顿
    rule3MinUtteranceLength: 10.0  // 最小音频长度
)
```

## 性能优化

### 1. 模型量化
使用 int8 量化模型减少内存占用：
```swift
let modelConfig = getZhZipformer20230615Int8()  // 使用量化版本
```

### 2. 多线程配置
```swift
let modelConfig = sherpaOnnxOnlineModelConfig(
    // ...
    numThreads: ProcessInfo.processInfo.processorCount
)
```

### 3. 音频缓冲优化
```swift
// 调整缓冲区大小以平衡延迟和性能
let bufferSize: AVAudioFrameCount = 512  // 较小的值降低延迟
```

## 故障排除

### 常见问题

1. **模型文件未找到**
   - 确保模型文件已添加到 Bundle Resources
   - 检查文件名是否正确

2. **音频权限问题**
   - 在 Info.plist 中添加麦克风权限：
   ```xml
   <key>NSMicrophoneUsageDescription</key>
   <string>需要麦克风权限进行语音识别</string>
   ```

3. **性能问题**
   - 使用量化模型
   - 调整线程数
   - 优化音频缓冲区大小

### 调试技巧

```swift
// 启用详细日志
#if DEBUG
 recognizer.setDebugMode(true)
#endif

// 监控识别状态
func checkRecognizerStatus() {
    print("isReady: \(recognizer.isReady())")
    print("isEndpoint: \(recognizer.isEndpoint())")
    print("numTrailingBlankFrames: \(recognizer.numTrailingBlankFrames())")
}
```

## 相关文档

- [iOS SwiftUI 集成](../ios-swiftui/CLAUDE.md)
- [iOS 示例](../ios/CLAUDE.md)
- [Sherpa-ONNX Swift API](https://k2-fsa.github.io/sherpa/onnx/swift-api/)
- [预训练模型](https://k2-fsa.github.io/sherpa/onnx/pretrained_models/index.html)

---

*更多信息和问题反馈请访问 Sherpa-ONNX GitHub：https://github.com/k2-fsa/sherpa-onnx*