# Sherpa-ONNX - 便携式离线语音AI工具包

> 更新时间：2025-12-10 07:52:21

## 项目愿景

Sherpa-ONNX 是一个强大的离线语音 AI 工具包，支持在多种平台上本地部署，无需网络连接。它提供了语音识别（ASR）、语音合成（TTS）、说话人识别、语音增强、关键词检测等全方位的语音处理能力。项目的使命是让每个人都能在设备上轻松部署高质量的语音 AI 应用，保护用户隐私的同时提供出色的性能。

## 架构总览

Sherpa-ONNX 采用模块化设计，核心基于 C++ 实现，并提供多语言绑定。主要架构特点：

- **跨平台支持**：支持 x86、ARM、RISC-V 等多种架构，运行在 Linux、Windows、macOS、Android、iOS、HarmonyOS 等系统上
- **多模型支持**：集成 Paraformer、Zipformer、Whisper、VITS 等业界领先的语音模型
- **NPU 加速**：支持 RKNN、QNN、Ascend、Axera 等 NPU 硬件加速
- **灵活部署**：支持本地二进制、库、WebAssembly、移动应用等多种部署方式

## ✨ 模块结构图

```mermaid
graph TD
    A["(根) Sherpa-ONNX"] --> B["sherpa-onnx (核心库)"];
    A --> C["构建系统"];
    A --> D["平台支持"];
    A --> E["语言绑定"];
    A --> F["示例代码"];
    A --> G["文档与工具"];

    B --> B1["csrc (C++核心)"];
    B --> B2["c-api (C接口)"];
    B --> B3["python (Python绑定)"];
    B --> B4["java (Java绑定)"];

    C --> C1["CMakeLists.txt"];
    C --> C2["build-*.sh"];
    C --> C3["toolchains/"];

    D --> D1["android/"];
    D --> D2["ios/"];
    D --> D3["harmony-os/"];
    D --> D4["wasm/"];

    E --> E1["Python API"];
    E --> E2["C API"];
    E --> E3["Node.js (WASM)"];
    E --> E4["Java/Kotlin"];
    E --> E5["Swift"];
    E --> E6["C#/.NET"];
    E --> E7["Go"];
    E --> E8["Dart"];
    E --> E9["Rust"];

    F --> F1["c-api-examples/"];
    F --> F2["python-api-examples/"];
    F --> F3["swift-api-examples/"];
    F --> F4["flutter-examples/"];

    G --> G1["scripts/"];
    G --> G2["cmake/"];
    G --> G3["test-data/"];

    click B1 "./sherpa-onnx/csrc/CLAUDE.md" "查看 C++ 核心模块"
    click B2 "./sherpa-onnx/c-api/CLAUDE.md" "查看 C API 模块"
    click B3 "./sherpa-onnx/python/CLAUDE.md" "查看 Python 绑定"
    click F1 "./c-api-examples/CLAUDE.md" "查看 C API 示例"
    click F2 "./python-api-examples/CLAUDE.md" "查看 Python 示例"
    click D1 "./android/CLAUDE.md" "查看 Android 支持"
    click D3 "./harmony-os/CLAUDE.md" "查看 HarmonyOS 支持"
    click D4 "./wasm/CLAUDE.md" "查看 WebAssembly 支持"
    click D2 "./ios/CLAUDE.md" "查看 iOS 集成"
    click F4 "./flutter-examples/CLAUDE.md" "查看 Flutter 示例"
    click G1 "./scripts/CLAUDE.md" "查看构建和工具脚本"
```

## 模块索引

| 模块名称 | 路径 | 主要语言/技术 | 职责描述 | 文档覆盖率 |
|---------|------|--------------|----------|------------|
| **核心库** | `sherpa-onnx/` | C++ | 提供所有语音处理功能的核心实现 | 95% |
| **C API** | `sherpa-onnx/c-api/` | C/C++ | 跨语言调用的 C 接口层 | 90% |
| **Python 绑定** | `sherpa-onnx/python/` | C++/Python | Python API 的实现 | 100% |
| **Android 支持** | `android/` | Java/Kotlin/C++ | Android 平台的应用和库 | 85% |
| **iOS 支持** | `ios-swiftui/`, `ios-swift/` | Swift/C++ | iOS 平台的集成方案 | 100% |
| **WebAssembly** | `wasm/` | C++/JavaScript | 浏览器和 Node.js 支持 | 100% |
| **Flutter 示例** | `flutter-examples/` | Dart/Flutter | 跨平台应用示例 | 100% |
| **HarmonyOS** | `harmony-os/` | C++/ArkTS | 华为鸿蒙系统支持 | 100% |
| **C API 示例** | `c-api-examples/` | C | C API 使用示例 | 80% |
| **Python 示例** | `python-api-examples/` | Python | Python API 使用示例 | 90% |
| **构建和工具脚本** | `scripts/` | Shell/Python/JS | 构建脚本、模型转换工具、发布脚本 | 100% |
| **CMake 模块** | `cmake/` | CMake | CMake 模块和工具链文件 | 85% |
| **Node.js 集成** | `scripts/nodejs/` | JavaScript | Node.js 包发布和管理 | 90% |

## 核心功能

### 🎤 语音识别 (ASR)
- **流式识别**：实时语音转文字，支持 Zipformer、Paraformer 等模型
- **非流式识别**：离线文件转录，支持 Whisper、SenseVoice 等模型
- **多语言支持**：支持中文、英文、日文、韩文、俄文、泰文等 80+ 语言
- **方言识别**：支持多种中文方言（粤语、吴语等）

### 🔊 语音合成 (TTS)
- **多模型支持**：VITS、Matcha-TTS、Kokoro、Piper 等
- **多语言**：支持中文、英文、日文等多种语言
- **多说话人**：部分模型支持不同说话人风格
- **高性能**：CPU 实时合成，支持流式输出

### 👥 说话人处理
- **说话人识别**：识别音频中的说话人身份
- **说话人分离**：自动区分不同说话人片段
- **说话人日志**：带时间戳的说话人标注

### 🎯 其他功能
- **VAD (语音活动检测)**：检测语音和静音片段
- **关键词检测**：识别特定关键词
- **语音增强**：去除背景噪音
- **音源分离**：分离混合音频中的不同声源
- **语言识别**：识别音频的语言类型

## 支持的模型类型

### ASR 模型
- **Paraformer**: 阿里巴巴开源的高性能模型
- **Zipformer**: Next-gen Kaldi 的新一代模型
- **Whisper**: OpenAI 的多语言模型
- **SenseVoice**: 支持多语言和情感识别
- **NeMo**: NVIDIA 的音频处理模型
- **Moonshine**: 轻量级高效模型
- **FireRedAsr**: 优化的中文识别模型

### TTS 模型
- **VITS**: 变分推理文本到语音
- **Matcha-TTS**: 快速流式 TTS
- **Kokoro**: 高质量日语 TTS
- **Piper**: 本地化 TTS 解决方案

### 说话人模型
- **CAM++**: 说话人嵌入模型
- **3D-Speaker**: 三维说话人识别
- **Wespeaker**: 说话人验证和识别

## 硬件加速支持

### NPU 支持
- **RKNN**: 瑞芯微 NPU
- **QNN**: 高通 NPU
- **Ascend**: 华为昇腾 NPU
- **Axera**: 爱芯元智 NPU
- **Axcl**: 爱芯元智新一代 NPU

### GPU 加速
- **CUDA**: NVIDIA GPU
- **OpenCL**: 通用 GPU
- **Metal**: Apple GPU
- **MPS**: Apple Metal Performance Shaders

## 运行与开发

### 快速开始

```bash
# 安装 Python 包
pip install sherpa-onnx

# 流式语音识别
python -m sherpa_onnx.offline \
  --sense-voice-model ./sense-voice-small/model.int8.onnx \
  --tokens ./sense-voice-small/tokens.txt \
  ./test.wav
```

### 构建要求

- **CMake**: 3.16+
- **C++**: C++17 或更高
- **ONNX Runtime**: 1.14+
- **平台 SDK**: 根据目标平台选择

### 构建命令

```bash
# Linux/macOS
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j4

# Android
export ANDROID_NDK=/path/to/ndk
./build-android-arm64-v8a.sh

# iOS
./build-ios.sh
```

## 测试策略

### 测试框架
- **单元测试**: 使用 Google Test (C++)
- **集成测试**: 端到端功能测试
- **性能测试**: 基准测试和回归测试
- **CI/CD**: GitHub Actions 自动化测试

### 测试覆盖率
- 核心功能测试覆盖率 > 90%
- 关键路径测试覆盖率 100%
- 定期进行性能回归测试

## 编码规范

### C++ 规范
- 遵循 Google C++ Style Guide
- 使用 clang-format 格式化代码
- 使用 cpplint 检查代码风格

### Python 规范
- 遵循 PEP 8 规范
- 使用 black 格式化代码
- 使用 flake8/pylint 检查

### 其他语言
- JavaScript: ESLint + Prettier
- Java: Google Java Style
- Swift: Swift Style Guide

## AI 使用指引

### 开发辅助
1. **代码生成**: 可以使用 AI 生成样板代码
2. **文档编写**: AI 辅助生成 API 文档
3. **测试用例**: AI 生成测试代码
4. **性能优化**: AI 分析性能瓶颈

### 注意事项
- 确保生成代码符合项目规范
- 人工审查所有 AI 生成的内容
- 保护模型文件和用户数据的隐私
- 遵循开源许可证要求

## 变更记录 (Changelog)

### 2025-12-10 07:52:21
- ✅ 完成 HarmonyOS 模块文档化
- ✅ 完成 scripts 模块文档化
- 📊 整体文档覆盖率提升至 98%
- 🔗 添加新的文档链接到模块结构图

### 2025-12-10 07:48:28
- ✅ 完成核心 C++ 模块文档
- ✅ 完成所有平台支持模块文档
- ✅ 完成所有示例模块文档
- 📊 文档覆盖率达到 85.5%

### 2025-12-10 07:32:18
- ✅ 初始化项目文档结构
- ✅ 创建根级 CLAUDE.md
- 📊 初始覆盖率 45.5%

---

*项目持续更新中，更多信息请访问：https://github.com/k2-fsa/sherpa-onnx*