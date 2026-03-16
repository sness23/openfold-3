# YouTube Demo Notes: What to Show

*Compiled March 2026*

---

## Quick-Start Demo (3 Commands)

The simplest possible demo — zero to structure prediction:

```bash
pip install openfold3
setup_openfold
run_openfold predict --query_json=examples/example_inference_inputs/query_ubiquitin.json
```

Ubiquitin is a small, well-known protein (~76 residues) that runs quickly. Experimental structure is PDB 1UBQ for comparison.

---

## Example Input Files

All in `examples/example_inference_inputs/`:

| File | What It Shows | Complexity |
|------|--------------|------------|
| `query_ubiquitin.json` | Single protein chain | Simplest |
| `query_homomer.json` | GCN4 leucine zipper homodimer (identical chains) | Simple |
| `query_multimer.json` | Complex multimer (PDB: 7cnx) with different chains | Medium |
| `query_protein_ligand.json` | Mcl-1 protein with ATP and a small molecule (SMILES) | Visually compelling |
| `query_protein_ligand_multiple.json` | Multiple ligand complexes | Medium |
| `query_single_protein_single_ligand.json` | Simple protein-ligand | Good for drug demo |
| `query_dna_ptm.json` | DNA with non-canonical residues | Shows DNA capability |

---

## Visual Demo Ideas

### 1. Predict a Protein and View It
- Run ubiquitin prediction
- Open output `.cif` in PyMOL or ChimeraX
- Color by pLDDT (B-factor) — blue = confident, red = uncertain
- Compare to experimental PDB structure (1UBQ) — overlay them

### 2. Drug Binding (Protein-Ligand)
- Run `query_protein_ligand.json` (Mcl-1 + ATP + small molecule)
- Show the SMILES string in the JSON input — "this is what a drug looks like to the model"
- Visualize the ligand sitting in the protein's binding pocket
- Explain this is how drug discovery works

### 3. Multiple Diffusion Samples
- Run with multiple seeds
- Show how diffusion generates slightly different structures
- Explain conformational sampling — the model isn't certain, it gives you options

### 4. The Diffusion Process (Conceptual)
- Show noise → structure animation (200 steps)
- "Like a sculptor revealing a statue from marble"

### 5. Confidence Scores
- Show the JSON confidence output
- Explain: pLDDT > 0.9 = trust this part, pLDDT < 0.5 = probably disordered
- Color the structure by confidence — immediately see which parts are reliable

---

## Command Variations for Demo

```bash
# Basic prediction (uses ColabFold MSA server by default)
run_openfold predict \
    --query-json examples/example_inference_inputs/query_ubiquitin.json \
    --output-dir ./demo_output/

# Protein-ligand prediction
run_openfold predict \
    --query-json examples/example_inference_inputs/query_protein_ligand.json \
    --output-dir ./demo_output/

# Low memory mode (for constrained GPUs)
run_openfold predict \
    --query-json examples/example_inference_inputs/query_ubiquitin.json \
    --runner-yaml examples/example_runner_yamls/low_mem.yml \
    --output-dir ./demo_output/

# Multi-GPU distributed inference
run_openfold predict \
    --query-json examples/example_inference_inputs/query_ubiquitin.json \
    --runner-yaml examples/example_runner_yamls/multiple_gpu.yml \
    --output-dir ./demo_output/

# Docker (no local install needed)
docker pull openfoldconsortium/openfold3:stable
docker run --gpus all -v ./data:/data openfoldconsortium/openfold3:stable \
    run_openfold predict --query-json /data/query.json --output-dir /data/output/
```

---

## Output Structure

```
output_dir/
├── query_name/
│   ├── seed_1/
│   │   ├── sample_0_model.cif          # 3D structure
│   │   ├── sample_0_confidences.json   # Per-atom scores (pLDDT, pDE)
│   │   ├── sample_0_confidences_aggregated.json  # Global metrics
│   │   ├── sample_1_model.cif
│   │   ├── ...
│   │   └── timing.json
│   ├── seed_2/
│   │   └── ...
│   ├── main/                           # Processed MSAs
│   └── paired/                         # Paired MSAs
├── inference_query_set.json
├── model_config.json
└── experiment_config.json
```

Default: 5 seeds x 5 diffusion samples = 25 candidate structures, ranked by `sample_ranking_score`.

---

## Existing Visualization Assets

In `assets/`:
- `protein_plot.png` — Example protein structure visualization
- `protein_ligand_plot.png` — Example protein-ligand complex visualization
- Benchmark comparison images

---

## Runner YAML Examples

In `examples/example_runner_yamls/`:

| File | Purpose |
|------|---------|
| `low_mem.yml` | Low memory mode (presets: predict, low_mem, pae_enabled) |
| `multiple_gpu.yml` | Distributed inference (devices: 4, num_nodes: 1) |
| `cuequivariance.yml` | GPU acceleration kernels |
| `save_msa_output.yml` | Persist MSA files between runs |
| `output_settings.yml` | Custom output formats (PDB vs CIF, embeddings) |
| `affinity.yaml` | Affinity calculations |

---

## Available Model Checkpoints

| Checkpoint | Description |
|---|---|
| `openfold3-p2-155k` | Default, recommended (155K training steps) |
| `openfold3-p2-145k` | Preview 2, earlier checkpoint |
| `openfold3-p1` | Preview 1, older |

---

## Setup Process (Good to Show)

`setup_openfold` is interactive:
1. Creates cache directory (`~/.openfold3` by default)
2. Downloads model parameters (~2GB)
3. Downloads Chemical Component Dictionary (CCD)
4. Optionally runs integration tests (~5 min on A100)
