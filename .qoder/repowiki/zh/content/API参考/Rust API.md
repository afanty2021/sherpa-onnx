# Rust API

<cite>
**本文档中引用的文件**   
- [README.md](file://README.md#L43)
- [rust-api-examples/README.md](file://rust-api-examples/README.md#L3)
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)
</cite>

## 目录
1. [简介](#简介)
2. [Rust API 概述](#rust-api-概述)
3. [FFI 边界与安全封装](#ffi-边界与安全封装)
4. [内存管理与生命周期](#内存管理与生命周期)
5. [错误处理](#错误处理)
6. [并发模型](#并发模型)
7. [零成本抽象与内存安全](#零成本抽象与内存安全)
8. [Cargo 配置与编译支持](#cargo-配置与编译支持)

## 简介

sherpa-onnx 是一个支持多种编程语言的语音处理库，其中包括对 Rust 语言的支持。然而，Rust 支持是由社区贡献和维护的，而不是项目核心团队直接开发的。根据项目文档，Rust 支持指向了外部仓库 thewh1teagle/sherpa-rs，这意味着 sherpa-onnx 项目本身并不包含原生的 Rust 代码实现。

**Section sources**
- [README.md](file://README.md#L43)
- [rust-api-examples/README.md](file://rust-api-examples/README.md#L3)

## Rust API 概述

由于 sherpa-onnx 项目本身不包含 Rust 源代码，其 Rust API 是通过社区维护的外部 crate（thewh1teagle/sherpa-rs）提供的。该 Rust crate 本质上是对 sherpa-onnx C API 的安全封装。C API 提供了丰富的语音处理功能，包括语音识别（ASR）、语音合成（TTS）、说话人识别、语音活动检测（VAD）等。

Rust API 的设计目标是为这些底层 C 功能提供安全、符合 Rust 习惯的接口。它通过 FFI（Foreign Function Interface）调用 C API，并在 Rust 层面处理内存管理、错误传播和生命周期等问题。主要功能模块包括：
- 在线和离线语音识别器
- 语音合成器
- 关键词检测器
- 语音活动检测器
- 说话人嵌入提取器
- 语音标记和标点添加

**Section sources**
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)

## FFI 边界与安全封装

Rust API 的核心是其对 C API 的安全封装。C API 定义了大量的结构体和函数，如 `SherpaOnnxOnlineRecognizerConfig`、`SherpaOnnxCreateOnlineRecognizer` 等。Rust 封装层负责将这些不安全的 C 接口转换为安全的 Rust 接口。

安全封装的关键在于管理资源的生命周期和所有权。C API 要求用户手动调用 `SherpaOnnxDestroyXXX` 系列函数来释放资源，这在 Rust 中是不安全的。Rust 封装通过实现 `Drop` trait 来自动管理这些资源，确保当 Rust 对象离开作用域时，相应的 C 资源会被正确释放，从而防止内存泄漏。

此外，Rust 封装还处理了字符串和数据缓冲区的转换。C API 使用 `const char*` 和原始指针，而 Rust API 使用 `String` 和 `Vec<f32>` 等安全类型，通过 `std::ffi::CString` 和 `std::slice::from_raw_parts` 等工具在两者之间进行安全转换。

**Section sources**
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)

## 内存管理与生命周期

Rust API 的内存管理完全遵循 Rust 的所有权和生命周期规则。所有通过 FFI 创建的资源都被包装在具有 `Drop` 实现的 Rust 结构体中。例如，`OnlineRecognizer` 结构体在 `Drop` 时会自动调用 `SherpaOnnxDestroyOnlineRecognizer`。

生命周期管理确保了引用的有效性。例如，`OnlineStream` 的生命周期必须短于其创建者 `OnlineRecognizer` 的生命周期，这通过 Rust 的借用检查器在编译时强制执行。这防止了悬空指针和使用已释放资源的错误。

对于从 C API 返回的指针（如识别结果），Rust 封装会立即将其复制到 Rust 的堆上（如 `String` 或 `Vec`），然后立即释放 C 端的内存。这确保了数据的所有权完全转移到 Rust，避免了跨 FFI 边界的复杂生命周期问题。

**Section sources**
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)

## 错误处理

Rust API 使用 `Result<T, E>` 类型进行错误处理，这与 C API 的返回码机制形成对比。C API 通常返回一个指针，`NULL` 表示错误，但不提供详细的错误信息。Rust 封装将这些错误转换为具体的错误类型，提供更丰富的上下文。

例如，创建识别器的函数可能会返回 `Result<OnlineRecognizer, CreationError>`。错误类型可以包含错误码、消息和可能的回溯信息。这种模式使得错误处理更加明确和安全，强制调用者处理可能的失败情况，而不是忽略返回值。

对于从 C API 返回的字符串结果，Rust 封装会检查指针是否为空，如果为空则返回一个错误，而不是尝试解引用一个空指针，从而避免了未定义行为。

**Section sources**
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)

## 并发模型

Rust API 的并发模型基于 C API 的线程安全性。C API 的设计允许在多个线程中并行处理多个流（`OnlineStream`）。Rust 封装通过实现 `Send` 和 `Sync` trait 来暴露这种能力。

`OnlineRecognizer` 通常可以被多个线程共享（`Sync`），因为它内部的状态是线程安全的。而 `OnlineStream` 通常只能在单个线程中使用（`!Sync`），但可以安全地在线程间转移所有权（`Send`）。这种设计允许用户在多线程环境中高效地处理多个并发的语音流。

Rust 的类型系统确保了对这些对象的并发访问是安全的。例如，编译器会阻止用户在多个线程中可变地借用同一个 `OnlineStream`，从而防止数据竞争。

**Section sources**
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)

## 零成本抽象与内存安全

Rust API 的目标是提供零成本抽象，即安全的 Rust 接口不会引入运行时开销。这通过在编译时进行所有安全检查来实现。`Drop` trait 的调用、借用检查和类型转换都是在编译时确定的，生成的代码与直接使用 C API 几乎一样高效。

内存安全是通过 Rust 的所有权系统保证的。所有对 C API 的调用都被封装在安全的抽象背后，防止了缓冲区溢出、空指针解引用和内存泄漏等常见错误。unsafe 代码块被严格限制在 FFI 调用的最小范围内，并由安全的 Rust 代码进行验证和保护。

**Section sources**
- [sherpa-onnx/c-api/c-api.h](file://sherpa-onnx/c-api/c-api.h)

## Cargo 配置与编译支持

Rust API 通过 Cargo 包管理器分发。用户可以通过在 `Cargo.toml` 中添加依赖来使用它。由于它依赖于 sherpa-onnx 的 C 库，构建过程需要正确配置链接器以找到 `libsherpa-onnx`。

对于 WASM 编译，项目提供了 WebAssembly 支持，允许在浏览器环境中使用语音功能。这通常涉及使用 Emscripten 将 C++ 代码编译为 WASM 模块，然后通过 JavaScript 胶水代码与 Rust 交互。

对于 `no_std` 环境的支持可能有限，因为语音处理通常需要动态内存分配和浮点运算，这些在 `no_std` 环境中可能不可用或需要特殊配置。Rust 封装本身可能依赖于标准库，因此直接在 `no_std` 环境中使用可能需要大量的修改和适配。

**Section sources**
- [README.md](file://README.md#L46)
- [wasm/CMakeLists.txt](file://wasm/CMakeLists.txt)