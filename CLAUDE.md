# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains ONNX-based inference implementations for HuggingFace encoder-decoder translation models **without PyTorch dependency**. It demonstrates how to run ORTModelForSeq2SeqLM models using native implementations in Python, C#, and Rust, eliminating the need for HuggingFace Optimum's PyTorch requirement.

The primary use case is Japanese-to-Korean translation using:
- **Encoder**: `cl-tohoku/bert-base-japanese-v2` (BERT Japanese tokenizer)
- **Decoder**: `skt/kogpt2-base-v2` (KoGPT2 tokenizer)
- **Model**: `sappho192/ffxiv-ja-ko-translator` (seq2seq ONNX model)

> **Note**: This repository has been merged into [sappho192/ffxiv-ja-ko-translator](https://github.com/sappho192/ffxiv-ja-ko-translator). See that repo for detailed guides and continued development.

## Directory Structure

```
onnx-hf-test/
├── python/          # Python implementation with ONNX Runtime
├── dotnet/          # C# implementation with native bindings
├── rust/            # Rust implementation
├── LICENSE
└── README.md
```

### Python Implementation (`python/`)

Contains Python scripts for ONNX inference:
- `infer_onnx.py`: Main inference script using Optimum's `ORTModelForSeq2SeqLM`
- `infer_torch.py`: Reference PyTorch implementation
- `download.py`: Downloads model files from HuggingFace
- `test_tokenizer.py`: Tokenizer testing utilities
- `exercise_onnx_greedy.py`: Greedy decoding experiments

**Dependencies**:
- `transformers`: Tokenizers (BertJapaneseTokenizer, PreTrainedTokenizerFast)
- `optimum`: ONNX Runtime integration (ORTModelForSeq2SeqLM)
- `onnxruntime`: Core ONNX inference engine
- `torch`: Tensor operations (for Optimum compatibility)

### C# Implementation (`dotnet/`)

Native C# implementation with minimal dependencies:
- `infer_onnx/`: Main C# project
  - `Program.cs`: Entry point for testing tokenizers and ONNX inference
  - `TestFunctions.cs`: Test suite for tokenization and model inference
  - `BertJapaneseTokenizer.dll`: Native tokenizer binding
  - `KoGPT2Tokenizer.cs`: Korean GPT2 tokenizer implementation
  - `NativeMethods.cs`: P/Invoke declarations for native libraries
  - `decoder.dll`: Native decoder library (compiled from Rust)
  - `vocab.txt`, `tokenizer.json`: Tokenizer vocabulary files
- `onnxtest.sln`: Visual Studio solution file

**Key Features**:
- Uses Microsoft.ML.OnnxRuntime for model inference
- Custom tokenizer implementations wrapped in C# classes
- Native library integration via P/Invoke
- No Python or PyTorch dependencies

### Rust Implementation (`rust/`)

Rust-based decoder implementation:
- `decoder/`: Rust library project
  - `src/`: Source code for decoder logic
  - `Cargo.toml`: Rust dependencies and build configuration
  - `build.rs`: Build script
  - `copy_libs.ps1`: PowerShell script to copy built libraries to dotnet project

**Purpose**: Provides native decoder library that can be called from C# via FFI.

## Commands

### Python

```bash
# Install dependencies
pip install transformers optimum onnxruntime torch pretty-downloader

# Download model files
python python/download.py

# Run ONNX inference
python python/infer_onnx.py

# Run PyTorch inference (reference)
python python/infer_torch.py

# Test tokenizers
python python/test_tokenizer.py
```

### C# (.NET)

```bash
# Build the project
cd dotnet
dotnet build onnxtest.sln

# Run inference tests
cd infer_onnx
dotnet run
```

**Prerequisites**:
- .NET SDK (6.0 or later recommended)
- Pre-built `decoder.dll` from Rust project

### Rust

```bash
# Build the decoder library
cd rust/decoder
cargo build --release

# Copy libraries to dotnet project (Windows PowerShell)
./copy_libs.ps1
```

**Output**: Creates `decoder.dll` (Windows) or equivalent shared library for other platforms.

## Architecture Notes

### Encoder-Decoder Pipeline

1. **Tokenization**: Input Japanese text is tokenized using BERT Japanese tokenizer
2. **Encoding**: ONNX encoder model processes token embeddings
3. **Decoding**: ONNX decoder generates Korean tokens autoregressively
4. **Detokenization**: Korean tokens decoded to text using KoGPT2 tokenizer

### ONNX Model Structure

The seq2seq model consists of separate ONNX graphs:
- `encoder_model.onnx`: Encodes source language input
- `decoder_model.onnx`: Generates target language output
- `decoder_with_past_model.onnx`: Decoder with KV cache for efficient generation
- `generation_config.json`: Generation hyperparameters

### Native Implementation Strategy

**Challenge**: HuggingFace Optimum requires PyTorch as a dependency ([Issue #526](https://github.com/huggingface/optimum/issues/526))

**Solution**:
1. Implement tokenizers natively in target language (C#/Rust)
2. Use ONNX Runtime directly for model inference
3. Manually handle encoder-decoder logic and generation loop
4. Manage KV cache for efficient autoregressive decoding

## Model Information

- **Base Model**: [sappho192/ffxiv-ja-ko-translator](https://huggingface.co/sappho192/ffxiv-ja-ko-translator)
- **Task**: Japanese-to-Korean translation
- **Domain**: Final Fantasy XIV game text
- **Encoder**: BERT Japanese (base, v2)
- **Decoder**: KoGPT2 (base, v2)

## Testing Examples

The `infer_onnx.py` includes test cases:

```python
texts = [
    "逃げろ!",  # Expected: "도망쳐!" (Run away!)
    "初めまして.",  # Expected: "반가워요" (Nice to meet you)
    "よろしくお願いします.",  # Expected: "잘 부탁드립니다." (Please treat me well)
    "ギルガメッシュ討伐戦",  # Expected: "길가메쉬 토벌전" (Gilgamesh raid)
    "ギルガメッシュ討伐戦に行ってきます。一緒に行きましょうか？",
    # Expected: "길가메쉬 토벌전에 갑니다. 같이 가실래요?" (Going to Gilgamesh raid. Want to come?)
    "夜になりました",  # Expected: "밤이 되었습니다" (It became night)
    "ご飯を食べましょう."  # Expected: "음, 이제 식사도 해볼까요" (Let's eat)
]
```

## Dependencies

### Python
- `transformers>=4.30.0`: Tokenizer implementations
- `optimum`: ONNX Runtime wrapper
- `onnxruntime`: ONNX inference engine
- `torch`: Required by Optimum
- `pretty-downloader`: File downloading utility

### C# (.NET)
- `Microsoft.ML.OnnxRuntime`: ONNX Runtime for .NET
- Custom native libraries: `decoder.dll`, `BertJapaneseTokenizer.dll`

### Rust
- Standard Rust dependencies via `Cargo.toml`
- Cross-platform shared library build toolchain

## Notes

- This project demonstrates **language-agnostic ONNX deployment** by implementing the same inference pipeline in Python, C#, and Rust
- The C# and Rust implementations are designed for **production deployment without Python runtime**
- ONNX models are stored with external data files for efficient loading
- The decoder library compiled from Rust provides reusable native inference logic
- For complete documentation and updates, refer to [ffxiv-ja-ko-translator](https://github.com/sappho192/ffxiv-ja-ko-translator)

## License

See LICENSE file for details.
