# iOS SwiftUI 集成

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / 平台支持 / iOS SwiftUI 集成

## 模块概述

本模块展示了如何使用 SwiftUI 框架在 iOS 平台上集成 Sherpa-ONNX。SwiftUI 是 Apple 推出的声明式 UI 框架，相比传统的 UIKit 更加简洁和现代化。

## 项目特点

- **SwiftUI 界面**：使用 SwiftUI 构建原生 iOS 应用界面
- **MVVM 架构**：采用 Model-View-ViewModel 设计模式
- **Combine 框架**：使用 Combine 进行响应式编程
- **实时语音处理**：支持实时语音识别和语音合成
- **多语言支持**：完整的多语言识别和合成功能

## 快速开始

### 1. 环境要求
- Xcode 14.0+
- iOS 15.0+
- Swift 5.7+

### 2. 打开项目
```bash
# 克隆项目
git clone https://github.com/k2-fsa/sherpa-onnx.git
cd sherpa-onnx/ios-swiftui

# 使用 Xcode 打开
open SherpaOnnxSwiftUI.xcodeproj
```

## 核心组件

### 1. ContentView.swift
主视图组件，包含：
- 语音识别界面
- 语音合成界面
- 模型配置
- 实时结果显示

### 2. ASRViewModel.swift
语音识别的视图模型：
- 音频录制管理
- 识别结果处理
- 状态管理

### 3. TTSViewModel.swift
语音合成的视图模型：
- 文本输入处理
- 语音生成
- 播放控制

### 4. AudioService.swift
音频服务封装：
- AVAudioEngine 管理
- 麦克风权限处理
- 音频格式转换

## 使用示例

### 基本语音识别界面

```swift
// ContentView.swift
import SwiftUI

struct ContentView: View {
    @StateObject private var asrViewModel = ASRViewModel()
    @StateObject private var ttsViewModel = TTSViewModel()
    @State private var selectedTab = 0

    var body: some View {
        TabView(selection: $selectedTab) {
            ASRView(viewModel: asrViewModel)
                .tabItem {
                    Image(systemName: "mic.fill")
                    Text("语音识别")
                }
                .tag(0)

            TTSView(viewModel: ttsViewModel)
                .tabItem {
                    Image(systemName: "speaker.wave.3.fill")
                    Text("语音合成")
                }
                .tag(1)

            SettingsView()
                .tabItem {
                    Image(systemName: "gearshape.fill")
                    Text("设置")
                }
                .tag(2)
        }
    }
}
```

### ASR 视图实现

```swift
// ASRView.swift
import SwiftUI
import Combine

struct ASRView: View {
    @ObservedObject var viewModel: ASRViewModel
    @State private var isRecording = false

    var body: some View {
        VStack(spacing: 20) {
            // 录音按钮
            Button(action: {
                if isRecording {
                    viewModel.stopRecording()
                } else {
                    viewModel.startRecording()
                }
                isRecording.toggle()
            }) {
                ZStack {
                    Circle()
                        .fill(isRecording ? Color.red : Color.green)
                        .frame(width: 100, height: 100)

                    Image(systemName: isRecording ? "stop.fill" : "mic.fill")
                        .font(.system(size: 40))
                        .foregroundColor(.white)
                }
            }

            // 状态指示
            Text(viewModel.status)
                .font(.caption)
                .foregroundColor(.secondary)

            // 识别结果
            ScrollView {
                Text(viewModel.recognitionText)
                    .font(.body)
                    .padding()
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .background(Color.gray.opacity(0.1))
                    .cornerRadius(10)
            }
            .frame(maxHeight: 200)

            Spacer()
        }
        .padding()
    }
}
```

### ViewModel 实现

```swift
// ASRViewModel.swift
import SwiftUI
import Combine
import AVFoundation

class ASRViewModel: NSObject, ObservableObject {
    @Published var recognitionText = ""
    @Published var status = "准备就绪"
    @Published var isRecording = false

    private var audioEngine: AVAudioEngine?
    private var recognizer: SherpaOnnxRecognizer?
    private var stream: SherpaOnnxStream?

    override init() {
        super.init()
        setupAudioSession()
        setupRecognizer()
    }

    private func setupAudioSession() {
        do {
            let audioSession = AVAudioSession.sharedInstance()
            try audioSession.setCategory(.record)
            try audioSession.setActive(true)
        } catch {
            print("音频会话设置失败: \(error)")
        }
    }

    private func setupRecognizer() {
        let modelConfig = getBilingualStreamZhEnZipformer20230220()
        let config = sherpaOnnxOnlineRecognizerConfig(
            modelConfig: modelConfig,
            enableEndpoint: true
        )

        recognizer = SherpaOnnxRecognizer(config: config)
    }

    func startRecording() {
        audioEngine = AVAudioEngine()
        guard let recognizer = recognizer else { return }

        stream = recognizer.createStream()
        status = "正在录音..."
        isRecording = true

        let inputNode = audioEngine!.inputNode
        let format = inputNode.outputFormat(forBus: 0)

        inputNode.installTap(onBus: 0, bufferSize: 1024, format: format) { [weak self] buffer, _ in
            guard let self = self,
                  let stream = self.stream else { return }

            let audio = buffer.audioBufferList.pointee.mBuffers.audioBuffer
            let samples = Array(UnsafeBufferPointer(start: audio.mData.assumingMemoryBound(to: Float.self),
                                                 count: Int(audio.mDataByteSize) / 4))

            stream.acceptWaveform(samples: samples)
            recognizer.decode(stream)

            if let result = recognizer.getResult(stream) {
                DispatchQueue.main.async {
                    self.recognitionText = result.text
                }
            }
        }

        do {
            try audioEngine?.start()
        } catch {
            print("启动音频引擎失败: \(error)")
        }
    }

    func stopRecording() {
        audioEngine?.stop()
        audioEngine?.inputNode.removeTap(onBus: 0)

        if let result = recognizer?.getResult(stream!) {
            recognitionText = result.text
        }

        status = "录音结束"
        isRecording = false
        stream = nil
    }
}
```

### 语音合成视图

```swift
// TTSView.swift
import SwiftUI

struct TTSView: View {
    @ObservedObject var viewModel: TTSViewModel
    @State private var text = ""
    @State private var selectedSpeaker = 0

    var body: some View {
        VStack(spacing: 20) {
            // 文本输入
            TextField("输入要合成的文本", text: $text)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .padding()

            // 说话人选择
            Picker("说话人", selection: $selectedSpeaker) {
                ForEach(0..<viewModel.speakerCount, id: \.self) { index in
                    Text("说话人 \(index)").tag(index)
                }
            }
            .pickerStyle(.segmented)
            .padding(.horizontal)

            // 生成按钮
            Button(action: {
                viewModel.generateSpeech(text: text, speakerId: selectedSpeaker)
            }) {
                HStack {
                    Image(systemName: "speaker.wave.3.fill")
                    Text("生成语音")
                }
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(10)
            }
            .disabled(text.isEmpty || viewModel.isGenerating)

            // 生成状态
            if viewModel.isGenerating {
                ProgressView()
                    .padding()
            }

            // 播放控制
            if viewModel.hasAudio {
                HStack {
                    Button(action: {
                        if viewModel.isPlaying {
                            viewModel.pausePlayback()
                        } else {
                            viewModel.startPlayback()
                        }
                    }) {
                        Image(systemName: viewModel.isPlaying ? "pause.fill" : "play.fill")
                            .font(.title)
                    }

                    Slider(value: $viewModel.playbackProgress, in: 0...1) { _ in
                        viewModel.seek(to: viewModel.playbackProgress)
                    }
                }
                .padding()
            }

            Spacer()
        }
        .onAppear {
            viewModel.setupTTS()
        }
    }
}
```

### 音频服务

```swift
// AudioService.swift
import AVFoundation

class AudioService: NSObject, ObservableObject {
    private var audioPlayer: AVAudioPlayer?
    @Published var isPlaying = false
    @Published var progress: Double = 0.0
    private var progressTimer: Timer?

    func playAudio(url: URL) {
        do {
            audioPlayer = try AVAudioPlayer(contentsOf: url)
            audioPlayer?.delegate = self
            audioPlayer?.play()
            isPlaying = true

            startProgressTimer()
        } catch {
            print("播放失败: \(error)")
        }
    }

    func pauseAudio() {
        audioPlayer?.pause()
        isPlaying = false
        stopProgressTimer()
    }

    private func startProgressTimer() {
        progressTimer = Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { _ in
            guard let player = self.audioPlayer else { return }
            self.progress = player.currentTime / player.duration
        }
    }

    private func stopProgressTimer() {
        progressTimer?.invalidate()
        progressTimer = nil
    }
}

extension AudioService: AVAudioPlayerDelegate {
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        isPlaying = false
        progress = 0.0
        stopProgressTimer()
    }
}
```

## 项目配置

### 1. Info.plist 权限配置

```xml
<key>NSMicrophoneUsageDescription</key>
<string>需要麦克风权限进行语音识别</string>
```

### 2. 模型文件管理

```swift
// ModelManager.swift
class ModelManager {
    static let shared = ModelManager()

    enum ModelType {
        case bilingual
        case english
        case chinese
    }

    func getModelConfig(type: ModelType) -> sherpaOnnxOnlineModelConfig {
        switch type {
        case .bilingual:
            return getBilingualStreamZhEnZipformer20230220()
        case .english:
            return getEnglishZipformer20230626()
        case .chinese:
            return getChineseZipformer20230615()
        }
    }
}
```

## 性能优化

### 1. 后台队列

```swift
class ASRViewModel: ObservableObject {
    private let audioQueue = DispatchQueue(label: "audio.processing")

    private func processAudio(buffer: AVAudioPCMBuffer) {
        audioQueue.async { [weak self] in
            // 在后台队列处理音频
            self?.recognizer?.decode(self!.stream!)
        }
    }
}
```

### 2. 内存管理

```swift
deinit {
    audioEngine?.stop()
    audioEngine = nil
    recognizer = nil
    stream = nil
}
```

## 测试

### 1. 单元测试

```swift
// ASRViewModelTests.swift
import XCTest
@testable import SherpaOnnxSwiftUI

class ASRViewModelTests: XCTestCase {
    var viewModel: ASRViewModel!

    override func setUp() {
        viewModel = ASRViewModel()
    }

    func testInitialValues() {
        XCTAssertFalse(viewModel.isRecording)
        XCTAssertEqual(viewModel.status, "准备就绪")
        XCTAssertEqual(viewModel.recognitionText, "")
    }
}
```

### 2. UI 测试

```swift
// SherpaOnnxSwiftUITests.swift
import XCTest

class SherpaOnnxSwiftUITests: XCTestCase {
    var app: XCUIApplication!

    override func setUp() {
        app = XCUIApplication()
        app.launch()
    }

    func testRecordingButton() {
        let recordButton = app.buttons["recordButton"]
        XCTAssertTrue(recordButton.exists)
        recordButton.tap()
        // 验证录音状态...
    }
}
```

## 构建和部署

### 1. 本地测试

```bash
# 在 Xcode 中选择模拟器或真机运行
Cmd + R
```

### 2. 创建发布版本

```bash
# 在 Xcode 中选择 "Any iOS Device"
Product -> Archive
```

### 3. App Store 分发

1. 创建 App Store Connect 应用
2. 上传构建版本
3. 提交审核

## 相关资源

- [SwiftUI 教程](https://developer.apple.com/tutorials/swiftui/)
- [Combine 框架](https://developer.apple.com/documentation/combine/)
- [AVAudioEngine](https://developer.apple.com/documentation/avaudioengine)
- [iOS Swift 集成](../ios-swift/CLAUDE.md)
- [iOS 集成示例](../ios/CLAUDE.md)

---

*更多信息和示例请参考项目源代码。*