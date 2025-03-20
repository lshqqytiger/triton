# AMD Perf Kernels

This directory contains customized/tuned/experimental kernels for AMD Instinct series GPUs.
Please make sure your Triton compiler is v2.1 or later, and is from the OpenAI Triton repository
[here](https://github.com/openai/triton). To install Triton, please see
[these](https://github.com/openai/triton/tree/main?tab=readme-ov-file#install-from-source) instructions.

## `06-fused-attention-transV.py`

This script is a copy of `tutorials/06-fused-attention.py` with the following
two changes:

- Tensor V is transposed in the way that seqlen/N_CTX dimension becomes the
fastest changing (a.k.a. leading or least strided) dimension.
This script produces better performance than `tutorials/06-fused-attention.py`
since it has better LDS access efficiency for tensor V.
Note that in the future, we'll improve the LDS access efficiency for
non-transposed tensor V, i.e. head dimension is the fastest changing dimension.
- Only fwd kernel is benchmarked.

## `06-fused-attention-fwd-transV.py`

This script is used to produce the best performance for fwd kernel.
It is a copy of `06-fused-attention-transV.py` with the following
changes:

- All bwd kernels are removed.
- Storing `m` at the end of the fwd kernel is removed.
- Autotuner is removed. All parameters for D=64 ad D=128 are pre-tuned
on MI250X and hard coded.

Note that this script is also used to benchmark FA performance with 2 GCDs.
Check the [2GCD benchmark script](https://github.com/ROCmSoftwarePlatform/triton/blob/triton-mlir/scripts/amd/benchmark_flash_attention.py) for more details.

## `flash-attention.py`

This script contains the Flash Attention kernel with the following support

- Arbitrary Q and KV sequence lengths, and arbitrary head sizes
- Autoregressive or "causal" masking
- Flash Attention v2 with variable sequence lengths
- Multi and Grouped Query attention
- ALiBi bias
- Matrix bias
- Persistent kernels. Useful when the sequence lengths are up to a moderate length and especially when doing causal attention.
- Int8 quantization

These are currently supported for the forward kernel only.

INT8 Quantization Support

1. <em>q_descale</em>, <em>k_descale</em>, and <em>v_descale</em> provided:
   - The first QK GEMM runs in INT8, then the output is dequantized to the specified <em>dtype</em>.
   - The second PV GEMM runs in the specified <em>dtype</em>.

2. <em>q_descale</em>, <em>k_descale</em>, <em>p_descale</em>, and <em>v_descale</em> provided:
   - Both the first and second GEMM operations run in INT8.
   - The results are dequantized to the specified <em>dtype</em> after both GEMMs.

3. Only <em>k_descale</em> and <em>v_descale</em> provided:
   - K and V are dequantized before the first and second GEMM operations, respectively.
   - Both GEMMs run in the specified <em>dtype</em>.

Note: The softmax operation is always performed in <em>fp32</em>.


## `06-attention-decode.py`

This contains the Flash Decoding kernel.

## `hbm-bw-test.py`

This is a script that measures HBM bandwidth performance on your device.

## `03-matrix-multiplication-all-types.py`

This script contains the GEMM kernel that supports int8, int32, fp16,
fp32, bf16 and f8 (both e5m2 and e4m3) datatypes.

## `03-matrix-multiplication-stream-k.py`

This script contains the GEMM kernel that implements [stream-k](https://arxiv.org/abs/2301.03598)

## `multreduce_matmul_kernel.py`

Kernel that implements GEMM with explicit multiply-reduce instructions for small block sizes. Such
small block sizes aren't natively supported by `tl.dot` operator.

Despite being numerically correct, this kernel performed worse than a corresponding GEMM kernel that
used `tl.dot` with minimum block size equal to $16$.

## `softmax.py`

Kernel that implements Softmax over a row of tensor.

## `rmsnorm.py`

Kernel that implements RMS Norm over a row of tensor.

## `layernorm.py`
Kernel that implements Layer Normalization over a row on tensor

## `fused_moe/moe-gemm.py`
Kernel that implements moe gemm.
