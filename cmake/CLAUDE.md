# CMake 模块

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / CMake 模块

## 模块概述

本目录包含 Sherpa-ONNX 项目使用的 CMake 模块和工具链文件。这些模块简化了依赖库的集成，提供了跨平台支持，并实现了自动化的依赖管理。

## 文件分类

### 🔧 依赖库模块
- **asio.cmake**: Asio 异步网络库
- **cargs.cmake**: CArgs 命令行参数解析
- **eigen.cmake**: Eigen 线性代数库
- **espeak-ng-for-piper.cmake**: eSpeak-ng 语音合成引擎
- **googletest.cmake**: Google Test 测试框架
- **hclust-cpp.cmake**: 层次聚类算法
- **kaldi-decoder.cmake**: Kaldi 解码器
- **kaldi-native-fbank.cmake**: Kaldi 特征提取
- **kaldifst.cmake**: Kaldi FST 库

### 🖥️ ONNX Runtime 模块

#### Linux ARM64
- **onnxruntime-linux-aarch64.cmake**: 动态链接版本
- **onnxruntime-linux-aarch64-static.cmake**: 静态链接版本
- **onnxruntime-linux-aarch64-gpu.cmake**: GPU 加速版本

#### Linux ARM
- **onnxruntime-linux-arm.cmake**: 动态链接版本
- **onnxruntime-linux-arm-static.cmake**: 静态链接版本

### 🛠️ 构建工具
- **cmake_extension.py**: CMake 扩展工具
- **__init__.py**: Python 包初始化

## 使用方法

### 1. 基本使用
在 CMakeLists.txt 中引用模块：

```cmake
# 设置 CMake 模块路径
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# 包含所需的模块
include(eigen)
include(onnxruntime-linux-x64)
```

### 2. 条件加载
根据平台自动选择合适的模块：

```cmake
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
        include(onnxruntime-linux-aarch64)
    elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "arm")
        include(onnxruntime-linux-arm)
    else()
        include(onnxruntime-linux-x64)
    endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    include(onnxruntime-win-x64)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    include(onnxruntime-macos-x64)
endif()
```

## 模块详解

### Eigen 线性代数库
```cmake
# eigen.cmake
find_package(Eigen3 QUIET)
if(NOT Eigen3_FOUND)
    # 如果系统没有安装，自动下载
    FetchContent_Declare(
        eigen
        URL https://gitlab.com/libeigen/eigen/-/archive/3.4.0.tar.gz
        URL_HASH SHA256=...
    )
    FetchContent_MakeAvailable(eigen)
endif()
```

### ONNX Runtime 集成
```cmake
# onnxruntime-linux-x64.cmake
include(FetchContent)

FetchContent_Declare(
    onnxruntime
    URL https://github.com/microsoft/onnxruntime/releases/download/v1.16.3/onnxruntime-linux-x64-1.16.3.tgz
    URL_HASH SHA256=...
)

# 创建导入目标
add_library(onnxruntime STATIC IMPORTED)
set_target_properties(onnxruntime PROPERTIES
    IMPORTED_LOCATION "${onnxruntime_SOURCE_DIR}/lib/libonnxruntime.so"
    INTERFACE_INCLUDE_DIRECTORIES "${onnxruntime_SOURCE_DIR}/include"
)
```

### Google Test 测试框架
```cmake
# googletest.cmake
option(SHERPA_ONNX_ENABLE_TESTS "Build tests" OFF)

if(SHERPA_ONNX_ENABLE_TESTS)
    include(FetchContent)
    FetchContent_Declare(
        googletest
        URL https://github.com/google/googletest/archive/03597a01ee50.tar.gz
        URL_HASH SHA256=...
    )

    # 禁用 Google Test 的安装规则
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(googletest)

    # 添加测试子目录
    enable_testing()
    add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()
```

## 构建选项

### 主要配置选项
```cmake
# Sherpa-ONNX 主 CMakeLists.txt 中的选项
option(SHERPA_ONNX_ENABLE_PYTHON "Build Python bindings" ON)
option(SHERPA_ONNX_ENABLE_JAVA "Build Java bindings" OFF)
option(SHERPA_ONNX_ENABLE_JNI "Build JNI interface" OFF)
option(SHERPA_ONNX_ENABLE_C_API "Build C API" ON)
option(SHERPA_ONNX_ENABLE_TESTS "Build tests" OFF)
option(SHERPA_ONNX_ENABLE_CHECK "Enable runtime checks" OFF)
option(SHERPA_ONNX_ENABLE_PORTAUDIO "Enable PortAudio" ON)
option(SHERPA_ONNX_ENABLE_WEBRTC "Enable WebRTC VAD" ON)
```

### 平台特定选项
```cmake
# Windows
if(WIN32)
    option(SHERPA_ONNX_ENABLE_MFC "Build MFC examples" OFF)
    option(SHERPA_ONNX_ENABLE_DIRECTML "Enable DirectML" OFF)
endif()

# macOS
if(APPLE)
    option(SHERPA_ONNX_ENABLE_ACCELERATE "Use Accelerate framework" ON)
    option(SHERPA_ONNX_ENABLE_COREML "Enable CoreML" OFF)
endif()

# Linux
if(UNIX AND NOT APPLE)
    option(SHERPA_ONNX_ENABLE_ALSA "Enable ALSA" ON)
    option(SHERPA_ONNX_ENABLE_PULSEAUDIO "Enable PulseAudio" ON)
endif()
```

## 交叉编译支持

### 工具链文件
```cmake
# toolchains/aarch64-linux-gnu.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)

set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)

set(CMAKE_FIND_ROOT_PATH /usr/aarch64-linux-gnu)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

### 使用示例
```bash
# ARM64 Linux 交叉编译
cmake -DCMAKE_TOOLCHAIN_FILE=toolchains/aarch64-linux-gnu.cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DANDROID_ABI=arm64-v8a \
      ..

# Android ARM64
cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=arm64-v8a \
      -DANDROID_PLATFORM=android-21 \
      ..
```

## 自定义 CMake 模块

### 创建新模块
```cmake
# mylibrary.cmake
# 查找库
find_path(MYLIB_INCLUDE_DIR
    NAMES mylibrary.h
    PATHS ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/mylibrary/include
)

find_library(MYLIB_LIBRARY
    NAMES mylibrary
    PATHS ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/mylibrary/lib
)

# 创建导入目标
if(MYLIB_INCLUDE_DIR AND MYLIB_LIBRARY)
    add_library(MyLibrary::MyLibrary INTERFACE IMPORTED)
    set_target_properties(MyLibrary::MyLibrary PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES ${MYLIB_INCLUDE_DIR}
        INTERFACE_LINK_LIBRARIES ${MYLIB_LIBRARY}
    )
endif()
```

### 在主项目中使用
```cmake
# 在 CMakeLists.txt 中
include(mylibrary)
target_link_libraries(your_target PRIVATE MyLibrary::MyLibrary)
```

## 性能优化

### 编译器优化
```cmake
# 根据构建类型设置编译选项
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR
       CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_options(your_target PRIVATE
            -O3
            -march=native
            -DNDEBUG
        )
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        target_compile_options(your_target PRIVATE
            /O2
            /arch:AVX2
            /DNDEBUG
        )
    endif()
endif()
```

### 链接时优化 (LTO)
```cmake
option(ENABLE_LTO "Enable Link Time Optimization" OFF)

if(ENABLE_LTO)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR
       CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -flto")
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /GL")
        set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /LTCG")
    endif()
endif()
```

## 调试和故障排除

### 详细输出
```bash
# CMake 配置时显示详细信息
cmake -DCMAKE_VERBOSE_MAKEFILE=ON -DCMAKE_LOG_LEVEL=DEBUG ..

# 构建时显示详细命令
make VERBOSE=1
```

### 检查依赖
```cmake
# 在 CMakeLists.txt 中添加诊断信息
message(STATUS "CMake version: ${CMAKE_VERSION}")
message(STATUS "System: ${CMAKE_SYSTEM_NAME}")
message(STATUS "Processor: ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID}")

# 打印所有缓存变量
get_cmake_property(_vars VARIABLES)
foreach(_var ${_vars})
    message(STATUS "${_var}=${${_var}}")
endforeach()
```

## 最佳实践

### 1. 现代化 CMake
```cmake
cmake_minimum_required(VERSION 3.16)

# 设置项目属性
project(SherpaOnnx VERSION 1.12.20 LANGUAGES CXX)

# 使用 C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 设置输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
```

### 2. 目标为中心的配置
```cmake
# 创建库目标
add_library(sherpa-onnx-core ${SOURCES})

# 设置目标属性
target_include_directories(sherpa-onnx-core
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_link_libraries(sherpa-onnx-core
    PUBLIC
        onnxruntime
        Eigen3::Eigen
    PRIVATE
        ${CMAKE_THREAD_LIBS_INIT}
)

# 编译选项
target_compile_features(sherpa-onnx-core PRIVATE cxx_std_17)
```

### 3. 安装和导出
```cmake
# 安装目标
install(TARGETS sherpa-onnx-core
    EXPORT SherpaOnnxTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
)

# 导出目标
install(EXPORT SherpaOnnxTargets
    FILE SherpaOnnxTargets.cmake
    NAMESPACE SherpaOnnx::
    DESTINATION lib/cmake/SherpaOnnx
)

# 创建配置文件
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    SherpaOnnxConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)
```

## 相关资源

- [CMake 官方文档](https://cmake.org/documentation/)
- [CMake 最佳实践](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html)
- [现代 CMake](https://cliutils.gitlab.io/modern-cmake/)
- [FetchContent 文档](https://cmake.org/cmake/help/latest/module/FetchContent.html)

---

*更多详细信息请参考各模块文件中的注释和文档。*