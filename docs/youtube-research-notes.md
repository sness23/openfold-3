# YouTube Video Research Notes: OpenFold3

*Compiled March 2026*

---

## The Story: From AlphaFold to OpenFold3

### The 50-Year Grand Challenge

Since Christian Anfinsen's 1972 Nobel Prize, scientists knew a protein's amino acid sequence determines its 3D shape, but couldn't predict how it folds. Experimental methods (X-ray crystallography, cryo-EM) take months to years per protein and cost tens of thousands of dollars each. Before AlphaFold, only ~17% of the ~20,000 human proteins had experimentally determined structures.

### Timeline

| Date | Event |
|------|-------|
| **December 2018** | **AlphaFold1 wins CASP13.** DeepMind enters the Critical Assessment of Structure Prediction competition and wins first place. Mohammed AlQuraishi (then at Harvard) writes his famous blog post "AlphaFold @ CASP13: What just happened?" describing a "broad sense of existential angst" among academic researchers. |
| **November 2020** | **AlphaFold2 wins CASP14.** Completely new architecture. Median GDT_TS of 92.4 — declared the protein folding problem "largely solved." AlQuraishi writes: "It feels like one's child has left home." |
| **July 2021** | **AlphaFold2 open-sourced** under Apache 2.0. Published in Nature. |
| **July 2022** | **200 million structures** released in the AlphaFold Protein Structure Database — nearly every known protein. |
| **May 2024** | **AlphaFold3 published** in Nature by Isomorphic Labs and DeepMind. Major leap: proteins + DNA + RNA + ligands. BUT: restrictive CC-BY-NC-SA 4.0 license. Controversy erupts — researchers accuse Nature of a double standard. |
| **October 2024** | **Nobel Prize in Chemistry** to Demis Hassabis and John Jumper (DeepMind) for AlphaFold, shared with David Baker for computational protein design. |
| **November 2024** | **AlphaFold3 code partially released** by DeepMind, but with restrictive non-commercial license. |
| **October 2025** | **OpenFold3-preview released** — first fully open-source reproduction, Apache 2.0. |
| **March 2026** | **OpenFold3-preview2** — competitive performance with AF3. Full training datasets released on AWS. |

### The Narrative Arc

A lone academic (AlQuraishi) watches an industrial lab (DeepMind) solve one of biology's greatest problems, writes a soul-searching blog post asking "What just happened?" — and then spends the next 7 years building the open-source alternative. When DeepMind reversed their AF2 openness with AF3's restrictive license, AlQuraishi and a coalition of pharma companies, tech giants, and academics built OpenFold3 to ensure this world-changing technology belongs to everyone.

---

## The People

### Mohammed AlQuraishi
- Assistant Professor, Department of Systems Biology, Columbia University (joined 2020)
- Undergraduate degrees in biology, computer science, and mathematics. MS in statistics, PhD in genetics from Stanford
- Before academia: founded two startups in mobile computing
- Wrote the pivotal blog posts reacting to AlphaFold1 and AlphaFold2
- Led original OpenFold (AF2 reproduction) and now OpenFold3

### The OpenFold Consortium
- Nonprofit hosted by the Open Molecular Software Foundation (OMSF)
- Mix of academic labs, pharma, biotech, and tech companies
- **Core members:** AlQuraishi Lab, Arzeda, Basecamp Research, Bayer, NVIDIA, UCB, Valence Labs, Dassault Systemes
- **Major pharma (joined 2025):** Bristol Myers Squibb, Novo Nordisk, Johnson & Johnson, Biogen
- **Tech:** SandboxAQ, Lambda, NVIDIA, AWS (provided cloud infrastructure)
- Training used 256 GPUs with AWS EC2 Capacity Blocks + Spot Instances for 85% cost savings

---

## The Science: Why Structure Prediction Matters

### What It Does
Proteins are chains of amino acids that fold into specific 3D shapes. The shape determines the function. Structure prediction takes a protein's sequence (a string of letters like MVLSPADKTNVKAAW...) and predicts the 3D coordinates of every atom.

### Why Biologists Care
- **Drug discovery:** Drugs work by binding to proteins. Knowing the 3D shape lets you design molecules that fit into binding pockets — like designing a key for a lock.
- **Understanding disease:** Misfolded proteins cause Alzheimer's, Parkinson's, cancer, and many metabolic diseases.
- **Vaccine development:** AF predictions helped understand SARS-CoV-2 viral protein structures during COVID.
- **Enzyme engineering:** Designing enzymes that decompose plastic, produce biofuels, or catalyze industrial reactions.
- **Neglected diseases:** Scientists used AF to find two existing FDA-approved drugs that could be repurposed for Chagas disease.

### Real-World Drug Discovery Impact
- Insilico Medicine: AI-designed fibrosis drug into human trials in under 18 months (vs. 4 years typical)
- Isomorphic Labs (DeepMind spinoff): raised $600 million (March 2025), preparing oncology clinical trials with AI-designed drugs
- Generate:Biomedicines: $1 billion partnership with Novartis
- AI drug discovery sector: $3.3 billion in venture funding in 2024
- FDA released draft guidance on AI for regulatory decision-making in 2025

---

## AlphaFold3 vs AlphaFold2: What Changed

### Architecture
| | AlphaFold2 | AlphaFold3/OpenFold3 |
|---|---|---|
| Core module | Evoformer (extensive MSA processing) | Pairformer (simpler, MSA processed separately) |
| Structure prediction | Structure Module (rigid body frames + torsion angles) | Diffusion model (raw atom coordinates from noise) |
| Output | Backbone frames + side chain angles | All-atom (x, y, z) coordinates directly |
| Post-processing | AMBER force field relaxation needed | No physics-based minimization |
| Molecules | Proteins only (single or multimer) | Proteins + DNA + RNA + ligands + ions |
| Behavior | Deterministic (one input = one output) | Generative (one input = distribution of structures) |

### Why Diffusion Was a Breakthrough
1. **Naturally handles uncertainty:** Disordered regions remain spread out, well-determined regions converge
2. **Eliminates complex equivariance machinery** (IPA attention, rigid body transforms) from AF2
3. **Extends to any molecule type** — just predict atom coordinates, no special parameterization needed
4. **Models conformational heterogeneity** — multiple valid structures for flexible regions

### The Diffusion Process (Simplified)
- **Training:** Take known structures from PDB. Add noise to atom positions. Train the model to predict the original clean coordinates from noisy ones.
- **Inference:** Start with pure random noise (a cloud of atoms). Iteratively denoise over 200 steps. Each step: predict clean coordinates, add back slightly less noise, repeat.
- **Multiple samples:** Generates 5 different structures per seed (different random starting noise), ranked by confidence scores.

---

## How OpenFold3 Works (Pipeline)

```
Sequence Input → Input Embedding → MSA Module (4 blocks)
                                         ↓
                              Pairformer (48 blocks)
                                         ↓
                              ← Recycle (3 times) ←
                                         ↓
                              Diffusion Module (24 blocks, 200 steps)
                                         ↓
                              3D Atomic Coordinates + Confidence Scores
```

**Simple analogy:** "The model reads the protein's sequence, looks at related sequences from evolution, builds a mental map of which parts should be close together, then sculpts the 3D structure atom by atom from a cloud of noise — like a sculptor revealing a statue from marble."

### Model Dimensions
- Single representation: 384 dims
- Pair representation: 128 dims
- Pairformer: 48 blocks, 4 attention heads
- Diffusion transformer: 24 blocks, 16 attention heads, 768 token dims
- Checkpoint size: ~2GB

---

## What OpenFold3 Takes as Input

JSON files specifying bioassemblies:

| Molecule Type | Input Format | Example |
|---|---|---|
| Protein | Amino acid sequence (1-letter codes) | `"MQIFVKTLTGKTITL..."` |
| DNA | Nucleotide sequence | `"AGCTAGCT"` |
| RNA | Nucleotide sequence | `"AGCUAGCU"` |
| Small molecule / Drug | SMILES string or CCD code | `"CC(=O)OC1..."` or `"ATP"` |

MSA options: ColabFold server (default), pre-computed MSAs, or MSA-free mode.

---

## What OpenFold3 Produces

Per prediction (default: 5 seeds x 5 samples = 25 candidate structures):

| Output | Description |
|---|---|
| `*_model.cif` | 3D structure file (mmCIF format) |
| `*_confidences.json` | Per-atom pLDDT and pDE scores |
| `*_confidences_aggregated.json` | Global metrics: avg_plddt, ptm, iptm, disorder, has_clash |
| `timing.json` | Runtime |

**Key confidence metrics:**
- **pLDDT** (0–1): Per-residue confidence. >0.9 = high confidence, <0.5 = likely disordered
- **pTM** (0–1): Global fold quality
- **ipTM** (0–1): Interface quality (how well chains interact)
- **sample_ranking_score**: `0.2*ptm + 0.8*iptm - 0.5*disorder - 100*has_clash`

---

## Competition Landscape

| Model | Developer | License | Key Differentiator |
|---|---|---|---|
| **AlphaFold3** | Google DeepMind / Isomorphic Labs | CC-BY-NC-SA 4.0 (restricted) | Gold standard accuracy, but closed |
| **OpenFold3** | AlQuraishi Lab / OpenFold Consortium | Apache 2.0 (fully open) | Full training stack; only model matching AF3 on monomeric RNA |
| **Boltz-1** | MIT Jameel Clinic | Open source, commercial | First fully commercially available open model at AF3-level accuracy |
| **Boltz-2** | MIT / Recursion | Open source | Uniquely predicts binding affinity (not just structure) |
| **Chai-1** | Chai Discovery | Open source | Adds protein language model embeddings |
| **Protenix** | ByteDance | Open source | Strong as a pose generator for virtual screening |
| **ESMFold** | Meta AI | Open source | No MSA required, faster but less accurate |

### Why OpenFold3 Stands Out
- **Only open model matching AF3 on RNA** — a standout result
- **Full training pipeline** — Boltz and Chai only release inference code
- **Apache 2.0** — most permissive license among AF3-class models
- **Full training data released** on AWS (300K experimental + 13M synthetic structures)

---

## Current Limitations

1. **Static structures only:** Predicts a frozen snapshot. Proteins are dynamic — they flex and change shape.
2. **Intrinsically disordered proteins:** ~30% of human proteome has no fixed structure. Models struggle here.
3. **Antibody-antigen prediction:** Still unreliable across all models. Critical for immunology and vaccine design. Biggest gap vs. AF3.
4. **Chirality and steric clashes:** ~4.4% of AF3 predictions have chirality mismatches or overlapping atoms.
5. **Binding affinity:** Most models predict where a drug sits, not how tightly it binds. (Boltz-2 is beginning to address this.)
6. **Conformational ensembles:** Single structure output, not the distribution of shapes a protein actually samples.
7. **Large complexes:** Accuracy drops for large multi-protein assemblies.

---

## Impact Numbers

| Metric | Value |
|---|---|
| Predicted structures (AlphaFold DB) | 200 million |
| Researchers using AlphaFold | 3+ million |
| Countries with users | 190+ |
| Human proteome coverage (before AF) | ~17% |
| Human proteome coverage (after AF) | ~98% |
| AI drug discovery VC funding (2024) | $3.3 billion |
| Isomorphic Labs fundraise (March 2025) | $600 million |
| Novartis-Generate:Bio partnership | $1 billion |
| OpenFold3 training structures | 300K experimental + 13M synthetic |
| OpenFold3 training GPUs | 256 A100s |

---

## Why Open Source Matters Here

**The licensing problem with AlphaFold3:**
- AlphaFold2 was Apache 2.0 (fully open). AlphaFold3 reversed this to CC-BY-NC-SA 4.0 (non-commercial only).
- Model weights require Google's explicit permission even for academic use.
- AlphaFold Server limited to 20 predictions/day.
- Isomorphic Labs uses AF3 internally for commercial drug discovery — competitive asymmetry.
- You cannot retrain AF3 on your own data, modify the architecture, or use it commercially.

**What OpenFold3 enables:**
- Apache 2.0 — unrestricted commercial and academic use
- Full training code — retrain from scratch or fine-tune on proprietary data
- Full training datasets released publicly on AWS
- Extensible architecture — researchers can modify and build on it
- Reproducibility — fundamental to science
- Pharma companies can customize for their specific drug targets

---

## Sources

- [AlQuraishi's "What Just Happened?" blog post (CASP13)](https://moalquraishi.wordpress.com/2018/12/09/alphafold-casp13-what-just-happened/)
- [AlQuraishi's "Child Has Left Home" blog post (CASP14)](https://moalquraishi.wordpress.com/2020/12/08/alphafold2-casp14-it-feels-like-ones-child-has-left-home/)
- [AlphaFold3 Nature paper](https://www.nature.com/articles/s41586-024-07487-w)
- [Nobel Prize in Chemistry 2024](https://www.nobelprize.org/prizes/chemistry/2024/press-release/)
- [OpenFold3 GitHub](https://github.com/aqlaboratory/openfold-3)
- [OpenFold3 March 2026 press release](https://www.businesswire.com/news/home/20260313170622/en/)
- [OpenFold Consortium new members (April 2025)](https://www.businesswire.com/news/home/20250415351561/en/)
- [AlphaFold: Five Years of Impact](https://deepmind.google/blog/alphafold-five-years-of-impact/)
- [Nature: Open-source protein AI aims to match AlphaFold](https://www.nature.com/articles/d41586-025-03546-y)
- [OpenFold3-preview2 technical report](https://portal.openfold.omsf.io/reports/of3p2_technical_report.pdf)
