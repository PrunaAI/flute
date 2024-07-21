<p align="center">
    <img src="assets/flute-logo.png" alt="" width="40%" align="top" style="border-radius: 10px; padding-left: 120px; padding-right: 120px; background-color: white;">
</p>

<p align="center">
  <em><strong>FLUTE</strong>: Flexible Lookup Table Engine for LUT-quantized LLMs <br></em>
</p>

<div align="center">

  ![GitHub License](https://img.shields.io/github/license/HanGuo97/flute)
  <a href="https://pypi.org/project/flute-kernel/">![Version](https://img.shields.io/pypi/v/flute-kernel)</a>
  <a href="https://arxiv.org/abs/2407.10960">![arXiv](https://img.shields.io/badge/arXiv-2407.10960-b31b1b.svg)</a>
</div>

<div align="center">

[[Introduction](#lookup-table-quantization)] [[Benchmarks](#benchmarks)] [[Model Zoo](#model-zoo)] [[Getting Started](#getting-started)] [[Compatibility](#support-and-compatibility)] 

</div>


# Lookup table quantization

- **Uniform quantization** converts full precision weights to lower-precision intervals of equal size.
- **Non-uniform quantization** maps weights to potentially _unequal_ intervals.
- **Lookup table (LUT) quantization** is a flexible variant of non-uniform quantization which can map intervals to arbitrary values via a lookup table.

Specifically, let $\mathbf{Q}$ denote the quantized weight, $\mathbf{s}$ the (group-wise) scales, and $\widehat{\mathbf{W}}$ the de-quantized weight,

<table align="center">
<tr>
<th>Uniform (Integer) Quantization</th>
<th>Lookup Table Quantization</th>
</tr>
<tr>
<td align="center">

$$\widehat{\mathbf{W}} = \mathtt{float}(\mathbf{Q}) \cdot \mathbf{s}$$

</td>
<td align="center">

$$\widehat{\mathbf{W}} = \mathtt{tableLookup}(\mathbf{Q}, \mathtt{table}) \cdot \mathbf{s}$$

</td>
</tr>
</table>

Examples of the lookup table suppored in FLUTE:

<table align="center">
<tr>
<th>Examples</th>
<th>Notes</th>
</tr>
<tr>
<td align="left">

`int4`, `int3`, `int2`

</td>
<td align="left">

recovers uniform/integer quantization

</td>
</tr>
<tr>
<td align="left">

`fp4`, `fp3`, `fp2`

</td>
<td align="left">
</td>
</tr>
<tr>
<td align="left">

`nf4`, `nf3`, `nf2`

</td>
<td align="left">

generalizes the `nf4` data-format introduced in QLoRA

</td>
</tr>
</td>
</tr>
<tr>
<td align="left">

any arbitrary table

</td>
<td align="left">

you could even learn it!

</td>
</tr>
</table>

### New Models Powered by FLUTE

As a proof of concept, we are releasing a few [models](#models) quantized using **Learned Normal Float (NFL)**. NFL is a simple extension to the `nf4` data format introduced in [QLoRA](https://arxiv.org/abs/2305.14314) with two key differences:
- NFL uses data to calibrate the Gaussian parameterization (currently only the `std` parameter, which could be subsumed into the scales).
- NFL supports bit widths beyond 4-bit.


# Benchmarks

<p align="center">
  <img src="assets/intro-figure.jpg" />
</p>


# Model Zoo

> [!NOTE]
> The models we release here are trained on more data and hence different from those in the paper.

> [!TIP]
> The HuggingFace Hub links are for `W4G64` quantization by default. To use the `W3G64` quantization, add `--revision nfl_w3g64`.

### [LLaMA-3 (8B)](https://huggingface.co/radi-cho/Meta-Llama-3-8B-FLUTE)


|             | Wiki | C4    | PIQA  | ARC-E | ARC-C | HellaSwag | Wino  | Avg.  |
| ----------- | ---- | ----- | ----- | ----- | ----- | --------- | ----- | ----- |
| Unquantized | 6.1  | 9.2   | 79.9  | 80.1  | 50.4  | 60.2      | 72.8  | 68.6  |
| W4G64       | 6.11 | 9.38  | 79.33 | 79.79 | 49.74 | 59.22     | 73.95 | 68.41 |
| W3G64       | 7.13 | 11.06 | 78.78 | 76.22 | 44.37 | 56.69     | 70.32 | 65.28 |

### [LLaMA-3 (70B)](https://huggingface.co/radi-cho/Meta-Llama-3-70B-FLUTE)


|             | Wiki | C4   | PIQA  | ARC-E | ARC-C | HellaSwag | Wino  | Avg.  |
| ----------- | ---- | ---- | ----- | ----- | ----- | --------- | ----- | ----- |
| Unquantized | 2.9  | 6.9  | 82.4  | 86.9  | 60.3  | 66.4      | 80.6  | 75.3  |
| W4G64       | 3.03 | 7.03 | 82.15 | 85.98 | 57.85 | 66.17     | 79.79 | 74.39 |
| W3G64       | 4.15 | 8.10 | 80.74 | 83.71 | 55.29 | 64.05     | 78.45 | 72.45 |


### [LLaMA-3 Instruct (8B)](https://huggingface.co/radi-cho/Meta-Llama-3-8B-Instruct-FLUTE)

|             | Wiki | C4    |
| ----------- | ---- | ----- |
| W4G64       | 6.78 | 10.61 |
| W3G64       | 7.75 | 12.28 |


### [LLaMA-3 Instruct (70B)](https://huggingface.co/radi-cho/Meta-Llama-3-70B-Instruct-FLUTE)

|       | Wiki | C4    |
| ----- | ---- | ----- |
| W4G64 | 3.67 | 7.95  |
| W3G64 | 4.90 | 10.86 |


### [Gemma-2 (9B)](https://huggingface.co/radi-cho/gemma-2-9b-FLUTE)

|             | Wiki | C4    | PIQA  | ARC-E | ARC-C | HellaSwag | Wino  | Avg.  |
| ----------- | ---- | ----- | ----- | ----- | ----- | --------- | ----- | ----- |
| Unquantized | 6.88 | 10.12 | 81.39 | 87.37 | 61.35 | 61.23     | 74.27 | 73.12 |
| W4G64       | 6.49 | 10.35 | 81.28 | 86.24 | 59.30 | 60.40     | 75.30 | 72.50 |


### Gemma-2 (27B)

Soon!

### [Gemma-2 Instruct (9B)](https://huggingface.co/radi-cho/gemma-2-9b-it-FLUTE)

|             | Wiki | C4    |
| ----------- | ---- | ----- |
| W4G64       | 6.88 | 11.02 |

### Gemma-2 Instruct (27B)

Soon!

# Getting Started

Install FLUTE with pip or [from source](#build-from-source):
```bash
# For CUDA 12.1
pip install flute-kernel
# For CUDA 11.8
pip install flute-kernel -i https://flute-ai.github.io/whl/cu118
```

## FLUTE + vLLM
FLUTE-quantized models ([Model Zoo](#models)) can be directly served using exisiting frameworks such as vLLM.

```diff
- python -m vllm.entrypoints.openai.api_server \
+ python -m flute.integrations.vllm vllm.entrypoints.openai.api_server \
    --model [MODEL] \
    --revision [REVISION] \
    --tensor-parallel-size [TP_SIZE] \
+   --quantization flute
```

For example, the following commmand runs the FLUTE-quantized LLaMA-3 (8B) on a single GPU.

```bash
python -m flute.integrations.vllm vllm.entrypoints.openai.api_server \
    --model radi-cho/Meta-Llama-3-8B-FLUTE \
    --quantization flute
```

We can then query the vLLM server as usual.
```bash
curl http://localhost:8000/v1/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "radi-cho/Meta-Llama-3-8B-FLUTE",
        "prompt": "San Francisco is a",
        "max_tokens": 7,
        "temperature": 0
    }'
```

## FLUTE + HuggingFace
FLUTE also runs out of the box with HuggingFace and its `accelerate` extension. This integration is mostly experimental and not optimized. Users sensitive to performance considerations should use the `vLLM` integration instead.

The following example performs simple quantization to the dense model. After this, the model can be used as normal. (Support for loading pre-quantized model is coming soon!)

```python
import flute.integrations.base
flute.integrations.base.prepare_model_flute(
    module=model.model.layers,  # for LLaMA-3 and Gemma-2
    num_bits=num_bits,
    group_size=group_size,
    fake=False,
    handle_hooks=True)  # for `accelerate` hooks
```



# Support and Compatibility

## Kernel

| Description      | Supported (via pip) | Supported (build from source) |
| ----------- | ----------- | ----------- |
| Input dtypes   | `torch.float16` `torch.bfloat16` |  |
| Bits | `4bit` `3bit` | `2bit` |
| Group Sizes | `32` `64` `128` `256` | ❓ |
| GPUs | `A100` `A6000` | `RTX 4090` `H100` (unoptimized) |

> [!WARNING]
> In the current release, we tuned the kernel templates for `torch.float16` only, and we noticed `torch.bfloat16` is slower than `torch.float16`. This likely because of lack of tuning, and that Ampere GPUs lack a hardware acceleration for `bfloat16` [vectorized atomic-add](https://github.com/HanGuo97/flute/blob/main/flute/csrc/cutlass_extensions_bf16.h#L27).

> [!WARNING]
> We noticed several numerically unstable situations using `bits=4, group-size=256, GPU=A100`. This is relatively rare (8 of 9360 test cases failed), but we suggest avoiding this particular use case for now. 

## Models

> [!NOTE]
> As of the current release, the kernel is shape-specialized due to legacy reasons (i.e., we tune tile sizes etc for each matrix shape). Please see the below chart for the supported use cases, as different platform and tensor parallel size changes the matrix shapes. We plan to add supports for a broad range of shapes in the near future. In the meantime, please let us know if you have any specific models in mind and we are happy to add support for them.

| Model      | Single GPU / Pipeline Parallel | Tensor Parallel |
| ----------- | ----------- | ----------- |
| LLaMA-3 (8B) | ✅ | |
| LLaMA-3 (70B) | ✅ | 2 or 4 GPUs  |
| Gemma-2 (9B) | ✅ |  |
| Gemma-2 (27B) | ✅ | 2 or 4 GPUs  |


## Quantizing Your Own Models

We provide two APIs to quantize a custom models. The easist way is to use the command line interface.

### Simple Normal Float Quantization

```bash
python -m flute.integrations.base \
    --pretrained_model_name_or_path meta-llama/Meta-Llama-3-70B-Instruct \
    --save_directory Meta-Llama-3-70B-Instruct-NF4 \
    --num_bits 4 \
    --group_size 128
```

The CLI essentially wraps around the following Python API,

```python
from transformers import (
    LlamaForCausalLM,
    Gemma2ForCausalLM,
    AutoModelForCausalLM)
import flute.integrations.base

model = AutoModelForCausalLM.from_pretrained(
    pretrained_model_name_or_path,
    device_map="cpu",
    torch_dtype="auto")

if isinstance(model, (LlamaForCausalLM, Gemma2ForCausalLM)):
    flute.integrations.base.prepare_model_flute(
        module=model.model.layers,
        num_bits=num_bits,
        group_size=group_size,
        fake=False)
else:
    # more models to come
    raise NotImplementedError
```

### Learned Normal Float Quantization

Soon!

# Build From Source

1. Clone the CUTLASS library.

```bash
# Unfortunately, the path is hard-coded as of now. If you install CUTLASS
# in a different directory, please make sure the corresponding path in
# `setup.py` is updated.
cd /workspace

git clone https://github.com/NVIDIA/cutlass.git
cd cutlass
git checkout v3.4.1
```

2. Build.

```bash
git clone https://github.com/HanGuo97/flute
cd flute
pip install -e .
```

**Note:** the build process requires having the local CUDA version (`nvcc --version`) match PyTorch's CUDA. In situations in which the build process throws an error related to CUDA version mismatch, try adding `--no-build-isolation`.


# Citation
To cite this repository:

```bibtex
@article{flute2024,
  title={Fast Matrix Multiplications for Lookup Table-Quantized LLMs},
  author={Guo, Han and Brandon, William and Cholakov, Radostin and Ragan-Kelley, Jonathan and Xing, Eric P and Kim, Yoon},
  journal={arXiv preprint arXiv:2407.10960},
  year={2024}
}
```
