# GPU Pricing & Hosted Services for OpenFold3

*Last updated: March 2026*

Running OpenFold3 inference requires a minimum of **32GB GPU VRAM** (recommended: A100 40GB+). This guide covers cloud GPU pricing and hosted alternatives.

---

## Cloud GPU Pricing (A100)

### AWS

AWS only offers A100s in 8-GPU bundles via **p4d.24xlarge** (8x A100 40GB, 96 vCPUs, 1152 GiB RAM).

| Type | Total (8 GPUs) | Per GPU/hr |
|------|----------------|------------|
| On-demand | ~$21.96/hr | ~$2.74 |
| Spot | ~$2.24–$17.50/hr (variable) | ~$0.28–$2.19 |
| 1yr Savings Plan | ~25–31% off on-demand | — |

AWS cut P4d prices by 33% in June 2025. Spot prices vary heavily by availability zone.

### Budget Providers

| Provider | GPU | On-Demand/hr | Notes |
|----------|-----|-------------|-------|
| **Vast.ai** | A100 40GB | ~$0.38–$0.72 | Marketplace model, prices fluctuate. Cheapest overall. |
| **Thunder Compute** | A100 40GB | ~$0.78 | Fixed pricing |
| **RunPod** | A100 80GB PCIe | $1.19 | $1.14 with 1yr commit |
| **RunPod** | A100 80GB SXM | $1.39 | $1.22 with 1yr commit |
| **Lambda Labs** | A100 80GB | ~$1.29–$2.06 | No egress fees |
| **JarvisLabs** | A100 80GB | ~$1.49 | — |

### Hyperscalers

| Provider | GPU | On-Demand/hr | Spot/Preemptible |
|----------|-----|-------------|-----------------|
| **Google Cloud** | A100 40GB | ~$1.15 | ~$0.10–$0.46 (60–91% off) |
| **Google Cloud** | A100 80GB | ~$1.57 | Similar discounts |
| **Azure** | A100 40GB | ~$3.40–$3.67 | ~$0.74 spot (NC24ads A100 v4) |
| **CoreWeave** | A100 80GB NVLink | ~$2.21 | Up to 60% off with commits |

### Cheapest Options Summary

**For a single inference job (a few hours):**
- **Vast.ai** marketplace: ~$0.38–$0.72/hr — cheapest but variable availability
- **Google Cloud Spot**: ~$0.10–$0.46/hr — cheapest from a hyperscaler, but can be preempted
- **Thunder Compute**: ~$0.78/hr — fixed price, no surprises

**For regular use:**
- **RunPod**: $1.14–$1.22/hr with 1yr commit, reliable
- **Lambda Labs**: ~$1.29/hr, good developer experience

---

## Hosted Structure Prediction Services

You don't necessarily need your own GPU. Several services host OpenFold3 or similar models.

### OpenFold3 Hosted

| Service | Price | Notes |
|---------|-------|-------|
| **NVIDIA NIM** | Free for prototyping | Production: ~$1/GPU/hr. Available at build.nvidia.com/openfold/openfold3 |
| **Neurosnap** | Free tier (credit-based) | Web interface at neurosnap.ai |
| **Tamarind Bio** | Free/Premium/Enterprise | Web interface at tamarind.bio/tools/openfold |

### AlphaFold3 (Google)

| Service | Price | Notes |
|---------|-------|-------|
| **AlphaFold Server** | Free (non-commercial) | 20 jobs/day, 5000 tokens/job. Cannot predict protein-drug interactions. alphafoldserver.com |

### Alternative Models (Similar Capabilities)

| Service | Model | Price | License |
|---------|-------|-------|---------|
| **Chai Discovery** | Chai-1 | Free web interface | Free for commercial drug discovery at lab.chaidiscovery.com |
| **Boltz** | Boltz-1/Boltz-2 | Free (open source, MIT) | Fully open, commercially usable. Also hosted on Neurosnap, Tamarind Bio |
| **Protenix** | Protenix | Free tier on Neurosnap | Open-source AF3 reproduction |

---

## Recommendations by Use Case

### "I just want to predict a few structures"
Use **Google AlphaFold Server** (free, non-commercial) or **Chai Discovery** web interface (free, including commercial). No setup required.

### "I want to run OpenFold3 specifically"
Use **NVIDIA NIM** (free for prototyping) or **Neurosnap** (free tier). For production, rent a GPU on **Vast.ai** or **RunPod** and use the OpenFold3 Docker image.

### "I need to run many predictions"
Rent an A100 on **RunPod** ($1.19/hr) or **Vast.ai** (~$0.50/hr). Use the Docker image:
```bash
docker pull ghcr.io/aqlaboratory/openfold-3:latest
```

### "I want to train or fine-tune"
Full training used 256 GPUs (32 nodes x 8). For fine-tuning, consider **CoreWeave** or **Lambda Labs** multi-GPU instances. AWS p4d.24xlarge spot instances can work but availability is unpredictable.

### "I'm on an RTX 2070 (8GB VRAM)"
You cannot run inference locally. Use any of the hosted services above — the free tiers on AlphaFold Server, Chai Discovery, or Neurosnap are your best bet.
