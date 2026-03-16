# GPU Requirements for OpenFold3

*Last updated: March 2026*

---

## Minimum Requirements

- **GPU VRAM**: 32GB minimum (A100 40GB recommended)
- **CUDA version**: 12.1 or higher
- **CUDA compute capability**: 8.0 or higher (Ampere and newer)

OpenFold3 is primarily tested on A100 40GB GPUs.

---

## What is CUDA Compute Capability?

CUDA compute capability is NVIDIA's version number for GPU hardware architecture. It determines what CUDA features and instructions the GPU supports. Each generation of NVIDIA GPUs has a specific compute capability:

| Compute Capability | Architecture | Example GPUs | Year |
|---|---|---|---|
| 7.0 | Volta | V100 | 2017 |
| 7.5 | Turing | RTX 2070, RTX 2080, T4 | 2018 |
| 8.0 | Ampere (data center) | A100, A30 | 2020 |
| 8.6 | Ampere (consumer) | RTX 3090, RTX 3080, A6000 | 2020 |
| 8.9 | Ada Lovelace | RTX 4090, RTX 4080, L40 | 2022 |
| 9.0 | Hopper | H100, H200 | 2022 |
| 12.0 | Blackwell | B200, GB200 | 2024 |

When PyTorch compiles CUDA kernels, it targets specific compute capabilities. OpenFold3's Dockerfile sets:

```
TORCH_CUDA_ARCH_LIST="8.0;8.6;9.0"
```

This means compiled binaries include optimized instructions only for A100, RTX 30-series, and H100 architectures. Running on an unsupported compute capability either fails or falls back to slower generic code. A separate Blackwell Dockerfile targets compute capability 12.0.

---

## GPU Compatibility Table

### Supported (32GB+ VRAM, compute capability 8.0+)

| GPU | VRAM | Compute Capability | Status |
|---|---|---|---|
| **A100 40GB** | 40GB | 8.0 | Official minimum, well-tested |
| **A100 80GB** | 80GB | 8.0 | Recommended for large complexes |
| **A6000** | 48GB | 8.6 | Should work, plenty of VRAM |
| **H100** | 80GB | 9.0 | Supported |
| **H200** | 141GB | 9.0 | Supported |
| **B200** | 192GB | 12.0 | Supported (Blackwell Dockerfile) |

### Marginal (supported compute capability, insufficient VRAM)

These GPUs have the right compute capability but less than 32GB VRAM. They may work with **low memory mode** on small inputs, but this is not tested or guaranteed.

| GPU | VRAM | Compute Capability | Notes |
|---|---|---|---|
| **RTX 3090** | 24GB | 8.6 | May work with low_mem preset for small proteins |
| **A30** | 24GB | 8.0 | Same as RTX 3090 |
| **RTX 3080** | 10GB | 8.6 | Almost certainly too little VRAM |

### Not Supported

| GPU | VRAM | Compute Capability | Why |
|---|---|---|---|
| **RTX 4090** | 24GB | 8.9 | Not in default arch list (would need recompilation), and 24GB is tight |
| **RTX 4080** | 16GB | 8.9 | Not in arch list, insufficient VRAM |
| **V100** | 32GB | 7.0 | Compute capability too old |
| **RTX 2080 Ti** | 11GB | 7.5 | Compute capability too old, insufficient VRAM |
| **RTX 2070** | 8GB | 7.5 | Compute capability too old, insufficient VRAM |
| **T4** | 16GB | 7.5 | Compute capability too old, insufficient VRAM |

---

## Low Memory Mode

OpenFold3 includes a `low_mem` preset that reduces GPU memory usage at the cost of speed. It works by:

- **Offloading** the MSA module and confidence heads to CPU
- **Processing samples sequentially** instead of batching (token and atom cutoffs set to 0)
- **Clearing CUDA cache** between inference steps

To use it, include `low_mem` in your model presets:

```yaml
model_update:
  presets:
    - predict
    - low_mem
    - pae_enabled
```

This mode is significantly slower but may allow GPUs with 24GB VRAM to run small predictions.

### Default Memory Cutoffs

| Parameter | Default | Low Memory Mode |
|---|---|---|
| `per_sample_token_cutoff` | 750 | 0 (always sequential) |
| `per_sample_atom_cutoff` | 10,000 | 0 (always sequential) |
| `token_cutoff` (confidence head) | 2,800 | 0 (always offload) |
| `clear_cache_between_steps` | false | true |
| `offload_inference.msa_module` | false | true |
| `offload_inference.confidence_heads` | true | true |

---

## Practical Recommendations

**Just want to predict a few structures?**
Use a hosted service instead of renting a GPU. See [gpu-pricing-and-hosting.md](gpu-pricing-and-hosting.md) for free options including NVIDIA NIM, Google AlphaFold Server, and Chai Discovery.

**Running inference regularly?**
Rent an A100 40GB from a cloud provider. Cheapest options start around $0.38–$1.19/hr. See [gpu-pricing-and-hosting.md](gpu-pricing-and-hosting.md) for pricing details.

**Training or fine-tuning?**
Full training used 256 A100 GPUs. Even fine-tuning requires multiple high-VRAM GPUs with DeepSpeed Stage 2.
