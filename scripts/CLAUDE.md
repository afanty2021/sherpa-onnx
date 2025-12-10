[根目录](../CLAUDE.md) > **scripts**

# 构建和工具脚本

> 最后更新：2025-12-10 07:52:21

## 模块职责

scripts 目录包含了 sherpa-onnx 项目的各种构建脚本、工具脚本和自动化脚本，主要职责包括：

- **跨平台构建**：提供 Linux、Windows、macOS、Android、iOS 等平台的构建脚本
- **APK 打包**：自动化 Android 应用的构建和打包流程
- **模型转换**：将各种格式的语音模型转换为 ONNX 格式
- **发布管理**：自动化多语言包的发布流程
- **工具辅助**：提供开发过程中的各种辅助工具

## 入口与启动

### 主要脚本分类

```
scripts/
├── apk/                    # Android APK 构建脚本
├── 3dspeaker/              # 3D Speaker 模型转换
├── bbpe/                   # BBPE 分词工具
├── ced/                    # CED 模型处理
├── dart/                   # Dart 包发布
├── dotnet/                 # .NET 绑定生成
├── go/                     # Go 包发布
├── node-addon-api/         # Node.js 插件构建
├── nodejs/                 # Node.js 包管理
├── [模型相关]/              # 各种模型的运行脚本
└── [工具脚本]/              # 其他辅助工具
```

### 使用方式

```bash
# 构建 APK（示例）
./scripts/apk/build-apk-asr.sh

# 模型转换（示例）
cd scripts/3dspeaker && ./run.sh

# 包发布（示例）
cd scripts/dart && ./release.sh
```

## 对外接口

### 1. APK 构建系统

#### build-apk-*.sh 脚本模板

```bash
#!/usr/bin/env bash
# 自动生成的 APK 构建脚本
# 使用前请设置 ANDROID_NDK 环境变量

set -ex
# 构建多架构版本
./build-android-arm64-v8a.sh
./build-android-armv7-eabi.sh
./build-android-x86-64.sh
./build-android-x86.sh

# 下载模型文件
model_name="sherpa-onnx-asr-model"
curl -SL -O https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/${model_name}.tar.bz2
tar xvf ${model_name}.tar.bz2

# 构建 APK
./gradlew assembleRelease
```

#### 支持的 APK 类型

- **ASR APK**: 语音识别应用
- **TTS APK**: 语音合成应用
- **VAD APK**: 语音活动检测
- **Speaker Diarization APK**: 说话人分离
- **Audio Tagging APK**: 音频标注
- **Keyword Spotting APK**: 关键词检测

### 2. 模型转换工具

#### 3D Speaker 转换

```bash
# 导出 ONNX 模型
python export-onnx.py \
  --model-path /path/to/3d-speaker \
  --output-dir ./onnx-models

# 测试转换结果
python test-onnx.py \
  --model-path ./onnx-models/model.onnx \
  --audio-file test.wav
```

#### BBPE 分词生成

```python
# 生成 BBPE 分词表
python generate_bbpe_table.py \
  --text-file corpus.txt \
  --vocab-size 1000 \
  --output bbpe.model
```

### 3. 包发布脚本

#### Dart 包发布

```bash
#!/bin/bash
# 自动发布 Dart 包到 pub.dev

# 更新版本号
sed -i "s/version: .*/version: $1/" pubspec.yaml

# 发布到 pub.dev
dart pub publish --dry-run
dart pub publish
```

#### Go 包发布

```bash
#!/bin/bash
# 发布 Go 模块

# 更新 go.mod
go mod tidy

# 创建 Git tag
git tag -a v$1 -m "Release version $1"
git push origin v$1
```

#### .NET 绑定生成

```powershell
# 生成 C# 绑定代码
./generate-bindings.ps1 \
  -input-file ../c-api/c-api.h \
  -output-dir ./generated \
  -namespace SherpaOnnx
```

### 4. Node.js 集成

```javascript
// Node.js 统一入口
const sherpa_onnx = require('./index.js');

// 创建识别器
const recognizer = sherpa_onnx.createOnlineRecognizer(config);

// 处理音频
const result = recognizer.decode(audioBuffer);
```

## 关键依赖与配置

### 1. 构建依赖

- **CMake**: 跨平台构建系统
- **Android NDK**: Android 原生开发
- **Xcode**: iOS/macOS 构建
- **Python**: 模型转换工具
- **Go**: Go 绑定开发
- **Dart SDK**: Flutter/Dart 开发
- **.NET SDK**: C# 开发

### 2. 环境变量配置

```bash
# Android 构建
export ANDROID_NDK=/path/to/android-ndk
export ANDROID_HOME=/path/to/android-sdk

# iOS 构建
export IOS_DEPLOYMENT_TARGET=12.0

# 通用配置
export SHERPA_ONNX_VERSION=1.12.19
```

### 3. 模型仓库配置

```json
{
  "model_repo": {
    "asr": "https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/",
    "tts": "https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/",
    "speaker": "https://github.com/k2-fsa/sherpa-onnx/releases/download/speaker-models/"
  }
}
```

## 测试与质量

### 测试脚本

- **check_style_cpplint.sh**: C++ 代码风格检查
- **test-*.sh**: 各模块功能测试
- **benchmark-*.sh**: 性能基准测试

### CI/CD 集成

```yaml
# .github/workflows/build.yml 示例
- name: Build Android APK
  run: |
    chmod +x ./scripts/apk/build-apk-asr.sh
    ./scripts/apk/build-apk-asr.sh

- name: Test Python bindings
  run: |
    cd ./python
    python -m pytest tests/
```

## 常见问题 (FAQ)

### Q1: 如何添加新的 APK 构建脚本？

A: 使用模板生成器：

```bash
python scripts/apk/generate-asr-apk-script.py \
  --model-list models.json \
  --output build-apk-new-asr.sh
```

### Q2: 模型转换失败怎么办？

A: 常见解决方案：
1. 检查 PyTorch 版本兼容性
2. 确保 ONNX 版本匹配
3. 验证输入模型格式
4. 查看转换日志中的错误信息

### Q3: 如何自定义构建选项？

A: 修改脚本中的环境变量：

```bash
export SHERPA_ONNX_ENABLE_TTS=OFF
export SHERPA_ONNX_ENABLE_VAD=ON
export SHERPA_ONNX_ENABLE_C_API=ON
```

### Q4: 发布包时版本号如何管理？

A: 使用自动化脚本：

```bash
# 更新所有包的版本
./scripts/update-version.sh 1.12.20

# 或使用 CMakeLists.txt 中的版本
export SHERPA_ONNX_VERSION=$(grep "SHERPA_ONNX_VERSION" ./CMakeLists.txt | cut -d '"' -f 2)
```

## 相关文件清单

### APK 构建相关

- `scripts/apk/build-apk-*.sh.in` - APK 构建脚本模板
- `scripts/apk/generate-*-apk-script.py` - 脚本生成器
- `scripts/apk/README.md` - APK 构建说明

### 模型转换相关

- `scripts/3dspeaker/export-onnx.py` - 3D Speaker 转换
- `scripts/3dspeaker/test-onnx.py` - 转换结果测试
- `scripts/bbpe/generate_bbpe_table.py` - BBPE 分词生成

### 包发布相关

- `scripts/dart/release.sh` - Dart 包发布
- `scripts/go/release.sh` - Go 包发布
- `scripts/dotnet/run.sh` - .NET 绑定生成

### Node.js 相关

- `scripts/nodejs/index.js` - Node.js 统一入口
- `scripts/node-addon-api/run.sh` - Node.js 插件构建

### 工具脚本

- `scripts/check_style_cpplint.sh` - 代码风格检查
- `scripts/export_bpe_vocab.py` - BPE 词汇导出
- `scripts/text2token.py` - 文本分词工具

### 模型运行脚本

- `scripts/sense-voice/run.sh` - SenseVoice 模型
- `scripts/moonshine/run.sh` - Moonshine 模型
- `scripts/wenet/run.sh` - WeNet 模型
- `scripts/vocos/run.sh` - Vocos 模型
- `scripts/gtcrn/run.sh` - GTCRN 模型

## 变更记录 (Changelog)

### 2025-12-10 07:52:21
- 📝 创建 scripts 模块文档
- 🔧 整理构建脚本分类
- 📋 添加常见问题解答
- 📊 完成文件清单整理

---

*注：使用脚本前请确保已正确配置所需的环境变量和依赖项。*