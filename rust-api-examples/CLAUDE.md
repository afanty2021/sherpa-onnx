# Rust API 示例

> 更新时间：2025-12-18 10:45:53

## 面包屑导航

[📁 sherpa-onnx](../CLAUDE.md) / API 示例 / Rust API 示例

## 模块概述

本模块是 Sherpa-ONNX Rust API 的示例入口。Rust 支持由社区贡献和维护，提供了 Rust 语言的绑定，使开发者能够在 Rust 应用中集成 Sherpa-ONNX 的强大语音处理功能。

## 重要说明

### 社区维护
Sherpa-ONNX 的 Rust 绑定由社区独立开发和维护，不是官方支持的一部分。如需使用，请访问 [sherpa-rs](https://github.com/thewh1teagle/sherpa-rs) 项目。

### 获取 Rust 绑定
```bash
# 克隆社区维护的 Rust 项目
git clone https://github.com/thewh1teagle/sherpa-rs.git
cd sherpa-rs
```

## 在 Rust 中使用 Sherpa-ONNX

### 1. 添加依赖
在 `Cargo.toml` 中添加：
```toml
[dependencies]
sherpa-rs = "0.1.0"  # 版本号请参考 sherpa-rs 项目
```

### 2. 基本使用示例
```rust
use sherpa_rs::{Recognizer, ModelConfig};

fn main() {
    // 配置模型
    let model_config = ModelConfig {
        encoder: "path/to/encoder.onnx".to_string(),
        decoder: "path/to/decoder.onnx".to_string(),
        joiner: "path/to/joiner.onnx".to_string(),
        tokens: "path/to/tokens.txt".to_string(),
        num_threads: 1,
    };

    // 创建识别器
    let recognizer = Recognizer::new(model_config).unwrap();

    // 识别音频
    let audio_file = "test.wav";
    let result = recognizer.recognize_file(audio_file).unwrap();

    println!("识别结果: {}", result.text);
}
```

### 3. 实时语音识别
```rust
use sherpa_rs::{OnlineRecognizer, Stream};

fn main() {
    // 创建在线识别器
    let recognizer = OnlineRecognizer::new(config).unwrap();

    // 创建音频流
    let mut stream = recognizer.create_stream();

    // 模拟实时音频输入
    loop {
        let audio_chunk = read_audio_chunk(); // 自定义音频读取

        if audio_chunk.is_empty() {
            break;
        }

        // 传入音频
        stream.accept_waveform(16000, &audio_chunk);

        // 解码
        recognizer.decode(&mut stream);

        // 获取中间结果
        let result = recognizer.get_result(&stream);
        if !result.is_empty() {
            println!("中间结果: {}", result);
        }
    }

    // 获取最终结果
    let final_result = recognizer.get_final_result(&stream);
    println!("最终结果: {}", final_result);
}
```

## Rust 优势

### 1. 内存安全
Rust 的所有权系统确保内存安全，无需垃圾回收，适合实时音频处理。

### 2. 零成本抽象
Rust 提供高性能，接近 C/C++ 的执行效率。

### 3. 并发安全
Rust 的并发模型让你能够安全地并行处理多个音频流。

### 4. 丰富的生态系统
- **音频处理**: cpal, rodio
- **序列化**: serde
- **异步**: tokio
- **网络**: reqwest

## 构建和运行

### 1. 安装 Rust
```bash
# 使用 rustup 安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
```

### 2. 创建新项目
```bash
cargo new sherpa_rust_example
cd sherpa_rust_example
```

### 3. 配置 Cargo.toml
```toml
[package]
name = "sherpa_rust_example"
version = "0.1.0"
edition = "2021"

[dependencies]
sherpa-rs = "0.1.0"
tokio = { version = "1.0", features = ["full"] }
cpal = "0.14"
anyhow = "1.0"
```

### 4. 运行示例
```bash
cargo run
```

## 高级用法

### 1. 异步处理
```rust
use tokio::task;
use sherpa_rs::Recognizer;

#[tokio::main]
async fn main() {
    let recognizer = Recognizer::new(config).unwrap();

    // 并发处理多个音频文件
    let files = vec!["audio1.wav", "audio2.wav", "audio3.wav"];

    let handles: Vec<_> = files
        .into_iter()
        .map(|file| {
            let recognizer = recognizer.clone();
            task::spawn_blocking(move || {
                recognizer.recognize_file(file)
            })
        })
        .collect();

    // 等待所有任务完成
    for handle in handles {
        let result = handle.await.unwrap().unwrap();
        println!("识别结果: {}", result.text);
    }
}
```

### 2. 实时音频处理
```rust
use cpal::{Device, Stream, StreamConfig};
use sherpa_rs::OnlineRecognizer;

fn setup_audio_stream() -> anyhow::Result<()> {
    let device = cpal::default_host().default_input_device()?;

    let config = StreamConfig {
        channels: 1,
        sample_rate: cpal::SampleRate(16000),
        buffer_size: cpal::BufferSize::Fixed(512),
    };

    let recognizer = OnlineRecognizer::new(config)?;

    let stream = device.build_input_stream(
        &config,
        |data: &[f32], _| {
            // 实时处理音频
            process_audio_chunk(&recognizer, data);
        },
        |err| eprintln!("错误: {}", err),
    )?;

    stream.play()?;
    Ok(())
}
```

## 性能优化

### 1. 使用 Rayon 并行处理
```toml
[dependencies]
rayon = "1.5"
```

```rust
use rayon::prelude::*;

fn batch_recognize(audio_files: Vec<String>) -> Vec<String> {
    audio_files
        .par_iter()
        .map(|file| recognizer.recognize_file(file))
        .collect()
}
```

### 2. 缓冲区复用
```rust
use std::collections::VecDeque;

struct AudioBuffer {
    buffers: VecDeque<Vec<f32>>,
}

impl AudioBuffer {
    fn get_buffer(&mut self) -> Vec<f32> {
        self.buffers.pop_front().unwrap_or_else(|| Vec::with_capacity(1600))
    }

    fn return_buffer(&mut self, mut buffer: Vec<f32>) {
        buffer.clear();
        self.buffers.push_back(buffer);
    }
}
```

## 故障排除

### 1. 编译错误
```bash
# 更新 Rust 工具链
rustup update stable

# 检查依赖版本
cargo tree
```

### 2. 运行时错误
确保模型文件路径正确：
```rust
use std::path::Path;

let model_path = Path::new("model");
assert!(model_path.exists(), "模型目录不存在");
```

### 3. 性能问题
- 使用 `--release` 模式编译
- 启用 LTO 优化
- 调整线程池大小

```toml
[profile.release]
lto = true
codegen-units = 1
panic = "abort"
```

## 相关资源

### Rust 音频库
- **cpal**: 音频 I/O
- **rodio**: 音频播放
- **hound**: WAV 文件读写
- **symphonia**: 音频解码

### Rust 并发库
- **tokio**: 异步运行时
- **rayon**: 数据并行
- **crossbeam**: 并发工具

### 官方资源
- [Rust 官网](https://www.rust-lang.org/)
- [Rust 文档](https://doc.rust-lang.org/)
- [sherpa-rs 项目](https://github.com/thewh1teagle/sherpa-rs)

---

*请注意：本模块只是 Rust 使用的入口点。完整的 Rust 绑定和示例请访问社区维护的 sherpa-rs 项目。*