# onnx-hf-test
Onnx utilization code without PyTorch dependency

> [!NOTE]  
> This repository has been merged into [[sappho192/ffxiv-ja-ko-translator](https://github.com/sappho192/ffxiv-ja-ko-translator)].  
> If you want to find get some details and guides regarding these code, please check the repository above.  

# Description
This repository has complete code which can run EncoderDecoder type translator model(ORTModelForSeq2SeqLM) without HuggingFace.Optimum or PyTorch dependency.  
Optimum provides ONNXRuntime-based inference but it currently depends on PyTorch [[Link](https://github.com/huggingface/optimum/issues/526)].  
So I implemented some required functions in C# or Rust to run my model without Python, which means anyone can run my translator in their own computer.  

Further description and updates will be on https://github.com/sappho192/ffxiv-ja-ko-translator .
