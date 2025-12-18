# MFC 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / MFC 示例

## 模块概述

本模块包含使用 Microsoft Foundation Class (MFC) 和 Visual C++ 开发的 Sherpa-ONNX 示例。这些示例展示了如何在传统的 Windows 桌面应用中集成语音处理功能，提供了图形用户界面，适合 Windows 平台的应用开发。

## 示例项目

### 1. 非流式语音识别
- **目录**: `NonStreamingSpeechRecognition`
- **功能**: 识别音频文件中的语音内容
- **预编译版本**: [x64](https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-non-streaming-asr-x64-v1.12.20.exe) / [x86](https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-non-streaming-asr-x86-v1.12.20.exe)

主要特性：
- 支持拖放音频文件
- 实时显示识别进度
- 支持多种音频格式 (WAV, MP3)
- 批量处理多个文件
- 导出识别结果

### 2. 流式语音识别
- **目录**: `StreamingSpeechRecognition`
- **功能**: 实时语音识别
- **预编译版本**: [x64](https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-streaming-asr-x64-v1.12.20.exe) / [x86](https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-streaming-asr-x86-v1.12.20.exe)

主要特性：
- 实时麦克风输入
- 实时显示识别结果
- 语音活动检测
- 自动断句
- 热词支持

### 3. 非流式文本转语音
- **目录**: `NonStreamingTextToSpeech`
- **功能**: 将文本转换为语音
- **预编译版本**: [x64](https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-non-streaming-tts-x64-v1.12.20.exe) / [x86](https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-non-streaming-tts-x86-v1.12.20.exe)

主要特性：
- 支持多语言文本
- 多说话人选择
- 音频参数调节
- 批量文本转换
- 保存为 WAV/MP3

## 环境要求

### 开发环境
- Windows 10 或更高版本
- Visual Studio 2022
- Windows 10 SDK
- CMake 3.16+

### 运行环境
- Windows 7 或更高版本
- Visual C++ Redistributable

## 快速开始

### 1. 使用预编译版本

直接下载并运行 EXE 文件：
```bash
# 下载流式语音识别
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.20/sherpa-onnx-streaming-asr-x64-v1.12.20.exe

# 运行
./sherpa-onnx-streaming-asr-x64-v1.12.20.exe
```

### 2. 从源码编译

#### 第一步：编译 Sherpa-ONNX
```bash
# 克隆项目
git clone https://github.com/k2-fsa/sherpa-onnx.git
cd sherpa-onnx

# 创建构建目录
mkdir build && cd build

# 配置 CMake
cmake -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_SHARED_LIBS=OFF \
      -DCMAKE_INSTALL_PREFIX=./install \
      ..

# 编译
cmake --build . --config Release --target install
```

#### 第二步：编译 MFC 示例
```bash
cd ../mfc-examples

# 使用 MSBuild 编译
msbuild ./mfc-examples.sln /p:Configuration=Release /p:Platform=x64

# 或在 Visual Studio 中打开解决方案
mfc-examples.sln
```

## 使用指南

### 非流式语音识别

1. **启动程序**
   - 运行 `NonStreamingSpeechRecognition.exe`

2. **加载模型**
   - 点击"模型"菜单，选择"加载模型"
   - 选择模型文件目录（包含 encoder.onnx, decoder.onnx, joiner.onnx, tokens.txt）

3. **识别音频**
   - 方法1：拖放音频文件到窗口
   - 方法2：点击"文件"→"打开"选择音频文件
   - 方法3：批量选择多个文件

4. **查看结果**
   - 识别结果实时显示在文本框中
   - 可以复制或保存结果

### 流式语音识别

1. **启动程序**
   - 运行 `StreamingSpeechRecognition.exe`

2. **设置模型**
   - 加载流式识别模型
   - 配置识别参数（端点检测、延迟等）

3. **开始识别**
   - 点击"开始"按钮开始录音
   - 对着麦克风说话
   - 识别结果实时显示

4. **设置热词**（可选）
   - 点击"热词"菜单
   - 添加自定义词汇和权重

### 文本转语音

1. **启动程序**
   - 运行 `NonStreamingTextToSpeech.exe`

2. **加载模型**
   - 选择 TTS 模型文件
   - 配置语音参数（语速、音调等）

3. **输入文本**
   - 在文本框中输入或粘贴文本
   - 支持中英文混合

4. **生成语音**
   - 点击"生成"按钮
   - 选择保存格式和位置

## 技术细节

### MFC 架构

```cpp
// 主对话框类
class CSpeechRecognitionDlg : public CDialog
{
    // 模型配置
    SherpaOnnxOfflineRecognizerConfig m_config;
    SherpaOnnxOfflineRecognizer* m_pRecognizer;

    // UI 控件
    CEdit m_editResult;
    CProgressCtrl m_progressBar;
    CButton m_btnStart;
};
```

### 音频处理

```cpp
// 音频文件读取
bool LoadAudioFile(const CString& filePath, std::vector<float>& samples)
{
    // 使用 Windows Media Foundation 或第三方库
    // 读取音频并转换为 float 格式
}

// 识别处理
void ProcessAudio(const std::vector<float>& samples)
{
    SherpaOnnxOfflineStream* stream = m_pRecognizer->CreateStream();
    stream->AcceptWaveform(samples.data(), samples.size());
    m_pRecognizer->Decode(stream);

    SherpaOnnxOfflineResult result = m_pRecognizer->GetResult(stream);
    UpdateUI(result.text);
}
```

### 实时音频捕获

```cpp
// 使用 WASAPI 捕获音频
class CAudioCapture
{
public:
    void StartCapture();
    void StopCapture();
    void SetCallback(std::function<void(const float*, int)> callback);
};
```

## 部署

### 创建安装包

使用 Inno Setup 创建安装程序：

```pascal
[Setup]
AppName=Sherpa-ONNX Speech Recognition
AppVersion=1.0
DefaultDirName={pf}\SherpaOnnx

[Files]
Source: "x64\Release\*.exe"; DestDir: "{app}"
Source: "models\*"; DestDir: "{app}\models"
Source: "msvcp140.dll"; DestDir: "{sys}"

[Icons]
Name: "{group}\Sherpa-ONNX ASR"; Filename: "{app}\NonStreamingSpeechRecognition.exe"
```

### 静态链接

```cmake
# 在 CMakeLists.txt 中
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
target_link_options(SherpaOnnx PRIVATE /STATIC)
```

## 性能优化

### 1. 使用多线程

```cpp
// 后台线程进行识别
UINT RecognitionThread(LPVOID param)
{
    CSpeechRecognitionDlg* pDlg = (CSpeechRecognitionDlg*)param;

    while (pDlg->m_bRunning)
    {
        // 处理音频队列
        ProcessAudioQueue();
        Sleep(10);
    }
    return 0;
}
```

### 2. 缓存模型

```cpp
class ModelCache
{
    static std::map<CString, SherpaOnnxOfflineRecognizer*> s_models;

public:
    static SherpaOnnxOfflineRecognizer* GetModel(const CString& modelPath)
    {
        if (s_models.find(modelPath) == s_models.end())
        {
            // 加载新模型
            s_models[modelPath] = new SherpaOnnxOfflineRecognizer(config);
        }
        return s_models[modelPath];
    }
};
```

## 故障排除

### 1. DLL 依赖问题
使用 Dependency Walker 检查依赖：
```cpp
// 在程序启动时检查
bool CheckDependencies()
{
    HMODULE hModule = LoadLibrary(L"onnxruntime.dll");
    if (!hModule)
    {
        AfxMessageBox(L"缺少 onnxruntime.dll");
        return false;
    }
    FreeLibrary(hModule);
    return true;
}
```

### 2. 音频设备问题
```cpp
// 列出可用音频设备
void ListAudioDevices()
{
    IMMDeviceEnumerator* pEnumerator = NULL;
    CoCreateInstance(__uuidof(MMDeviceEnumerator), NULL, CLSCTX_ALL,
                    __uuidof(IMMDeviceEnumerator), (void**)&pEnumerator);

    // 枚举设备...
}
```

### 3. 模型加载失败
```cpp
// 验证模型文件
bool ValidateModelFiles(const CString& modelDir)
{
    CString encoderPath = modelDir + L"\\encoder.onnx";
    if (!PathFileExists(encoderPath))
    {
        AfxMessageBox(L"找不到 encoder.onnx");
        return false;
    }
    // 检查其他文件...
    return true;
}
```

## 扩展开发

### 1. 添加新功能
- 语音命令控制
- 实时字幕
- 多语言切换
- 用户配置保存

### 2. 界面美化
- 使用现代化的 UI 库（如 Ribbon UI）
- 添加皮肤支持
- 响应式布局

### 3. 插件系统
```cpp
// 定义插件接口
class ISherpaPlugin
{
public:
    virtual void Initialize() = 0;
    virtual void ProcessAudio(float* samples, int count) = 0;
    virtual void Release() = 0;
};
```

## 相关资源

- [Visual Studio 2022](https://visualstudio.microsoft.com/vs/)
- [MFC 文档](https://docs.microsoft.com/cpp/mfc/)
- [Windows Audio APIs](https://docs.microsoft.com/en-us/windows/win32/coreaudio/)
- [Sherpa-ONNX 下载页面](https://github.com/k2-fsa/sherpa-onnx/releases)

---

*更多问题请参考项目 README 或提交 Issue。*