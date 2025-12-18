# 交叉编译工具链

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / 交叉编译工具链

## 模块概述

本目录包含 Sherpa-ONNX 项目的交叉编译工具链文件。这些文件允许您在一个平台上为另一个不同的平台（不同的架构或操作系统）编译代码，实现对各种嵌入式设备、移动平台和专用硬件的支持。

## 支持的工具链

### ARM 平台
- **aarch64-linux-gnu.toolchain.cmake**: ARM64 Linux 交叉编译
- **arm-linux-gnueabihf.toolchain.cmake**: ARM Linux（硬浮点）交叉编译

### RISC-V 平台
- **riscv64-linux-gnu.toolchain.cmake**: RISC-V 64 Linux 交叉编译
- **riscv64-linux-gnu-spacemit.toolchain.cmake**: Spacemit RISC-V 64 专用工具链

### iOS 平台
- **ios.toolchain.cmake**: iOS 交叉编译（支持 arm64, x86_64）

## 使用方法

### 1. 基本交叉编译

```bash
# 为 ARM64 Linux 编译
cmake -DCMAKE_TOOLCHAIN_FILE=toolchains/aarch64-linux-gnu.toolchain.cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=./install \
      ..

# 为 RISC-V 64 Linux 编译
cmake -DCMAKE_TOOLCHAIN_FILE=toolchains/riscv64-linux-gnu.toolchain.cmake \
      -DCMAKE_BUILD_TYPE=Release \
      ..

# 为 iOS 编译
./build-ios.sh  # 使用预定义脚本
```

### 2. 设置编译器和工具

```cmake
# 在工具链文件中定义
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)

# 设置编译器
set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)

# 设置查找路径
set(CMAKE_FIND_ROOT_PATH /usr/aarch64-linux-gnu)
```

## 工具链详解

### ARM64 Linux 工具链
```cmake
# aarch64-linux-gnu.toolchain.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)

# 交叉编译工具链前缀
set(CROSS_COMPILE aarch64-linux-gnu-)

# 设置编译器
set(CMAKE_C_COMPILER ${CROSS_COMPILE}gcc)
set(CMAKE_CXX_COMPILER ${CROSS_COMPILE}g++)
set(CMAKE_AR ${CROSS_COMPILE}ar)
set(CMAKE_STRIP ${CROSS_COMPILE}strip)
set(CMAKE_RANLIB ${CROSS_COMPILE}ranlib)

# 设置根文件系统路径
set(CMAKE_FIND_ROOT_PATH /usr/${CROSS_COMPILE})

# 搜索程序的模式
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

# 设置编译标志
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=armv8-a")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=armv8-a")
```

### RISC-V 64 工具链
```cmake
# riscv64-linux-gnu.toolchain.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR riscv64)

set(CROSS_COMPILE riscv64-linux-gnu-)

set(CMAKE_C_COMPILER ${CROSS_COMPILE}gcc)
set(CMAKE_CXX_COMPILER ${CROSS_COMPILE}g++)

# RISC-V 特定标志
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=rv64gc -mabi=lp64d")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=rv64gc -mabi=lp64d")
```

### iOS 工具链
```cmake
# ios.toolchain.cmake
# 这个文件较大，支持多个 iOS 架构和版本

# 设置平台
set(PLATFORM IOS)
set(ARCHS arm64;x86_64)

# iOS SDK 路径
set(CMAKE_OSX_SYSROOT iphoneos)
set(CMAKE_OSX_DEPLOYMENT_TARGET 11.0)

# 编译器设置
set(CMAKE_C_COMPILER xcrun -sdk iphoneos clang)
set(CMAKE_CXX_COMPILER xcrun -sdk iphoneos clang++)
```

## 安装交叉编译工具链

### Ubuntu/Debian

```bash
# ARM64 工具链
sudo apt-get install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

# ARM 工具链
sudo apt-get install gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf

# RISC-V 工具链
sudo apt-get install gcc-riscv64-linux-gnu g++-riscv64-linux-gnu
```

### CentOS/RHEL

```bash
# 启用 EPEL 仓库
sudo yum install epel-release

# 安装交叉编译器
sudo yum install aarch64-linux-gnu-gcc-c++
sudo yum install gcc-riscv64-linux-gnu
```

### macOS

```bash
# 使用 Homebrew
brew install aarch64-linux-gnu-binutils
brew install riscv64-elf-gcc

# 或从源码编译
```

### 从源码编译工具链

```bash
# 下载并编译 GCC 交叉编译器
wget https://ftp.gnu.org/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz
tar xf gcc-12.2.0.tar.gz
cd gcc-12.2.0

# 下载依赖
./contrib/download_prerequisites

# 配置
mkdir build && cd build
../configure --target=aarch64-linux-gnu \
            --prefix=/opt/cross \
            --disable-multilib \
            --enable-languages=c,c++

# 编译和安装
make -j$(nproc)
sudo make install
```

## 编译示例

### 1. 为 ARM64 Linux 编译

```bash
# 创建构建目录
mkdir build-arm64 && cd build-arm64

# 配置
cmake -DCMAKE_TOOLCHAIN_FILE=../toolchains/aarch64-linux-gnu.toolchain.cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DSHERPA_ONNX_ENABLE_PYTHON=OFF \
      -DSHERPA_ONNX_ENABLE_JNI=OFF \
      -DCMAKE_INSTALL_PREFIX=./install \
      ..

# 编译
make -j$(nproc)

# 安装
make install
```

### 2. 为 RISC-V 64 编译

```bash
mkdir build-riscv64 && cd build-riscv64

cmake -DCMAKE_TOOLCHAIN_FILE=../toolchains/riscv64-linux-gnu.toolchain.cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DSHERPA_ONNX_ENABLE_PYTHON=OFF \
      -DSHERPA_ONNX_ENABLE_JAVA=OFF \
      ..

make -j$(nproc)
```

### 3. 使用脚本编译

```bash
# 使用项目提供的脚本
./build-android-arm64-v8a.sh
./build-aarch64-linux-gnu.sh
./build-riscv64-linux-gnu.sh
```

## 高级配置

### 1. 自定义工具链

```cmake
# 创建自定义工具链文件 my-custom.toolchain.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR myarch)

# 设置编译器路径
set(CMAKE_C_COMPILER /opt/cross/bin/myarch-gcc)
set(CMAKE_CXX_COMPILER /opt/cross/bin/myarch-g++)

# 设置标志
set(CMAKE_C_FLAGS "-march=myarch")
set(CMAKE_CXX_FLAGS "-march=myarch")

# 包含必要的库
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static")
```

### 2. 多架构构建

```bash
# 创建不同的构建目录
mkdir build-arm64 build-x86_64

# 并行构建
cmake -S . -B build-arm64 -DCMAKE_TOOLCHAIN_FILE=toolchains/aarch64-linux-gnu.toolchain.cmake &
cmake -S . -B build-x86_64 -DCMAKE_TOOLCHAIN_FILE=toolchains/x86_64-linux-gnu.toolchain.cmake &

wait

# 并行编译
make -C build-arm64 -j$(nproc) &
make -C build-x86_64 -j$(nproc) &
wait
```

## 故障排除

### 1. 找不到交叉编译器

```bash
# 检查编译器是否在 PATH 中
which aarch64-linux-gnu-gcc

# 添加到 PATH
export PATH=/opt/cross/bin:$PATH
```

### 2. 链接错误

```cmake
# 在工具链文件中添加库路径
set(CMAKE_LIBRARY_PATH /opt/cross/lib)
set(CMAKE_INCLUDE_PATH /opt/cross/include)
```

### 3. 运行时问题

```bash
# 使用 qemu-user-static 测试
sudo apt-get install qemu-user-static

# 运行 ARM64 二进制
qemu-aarch64-static ./your-program
```

## 部署到目标设备

### 1. 复制文件

```bash
# 使用 scp 复制到目标设备
scp -r install/* user@target-device:/opt/sherpa-onnx/

# 或使用 rsync
rsync -avz install/ user@target-device:/opt/sherpa-onnx/
```

### 2. 创建安装包

```bash
# 使用 CPack
cpack -G TGZ  # 创建 tar.gz 包
cpack -G DEB  # 创建 Debian 包
cpack -G RPM  # 创建 RPM 包
```

### 3. Docker 容器

```dockerfile
FROM arm64v8/ubuntu:22.04

COPY --from=builder /opt/sherpa-onnx /opt/sherpa-onnx
ENV PATH=/opt/sherpa-onnx/bin:$PATH

CMD ["sherpa-onnx"]
```

## 相关资源

- [CMake 交叉编译文档](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html)
- [GCC 交叉编译器](https://gcc.gnu.org/wiki/CrossCompilerHOWTO)
- [LLVM 交叉编译](https://clang.llvm.org/docs/CrossCompilation.html)
- [Yocto Project](https://www.yoctoproject.org/docs/)

---

*更多高级配置请参考各个工具链文件中的注释。*