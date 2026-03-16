# Demo Runbook: Exact Commands for Recording

*Step-by-step instructions for the on-camera demo segment*

---

## Pre-Recording Setup

Do all of this BEFORE you hit record. You don't want to wait for downloads on camera.

### 1. Get a GPU

Rent an A100 40GB (Vast.ai, RunPod, or similar). Make sure CUDA 12.1+ is installed.

### 2. Install and Setup

```bash
# Install OpenFold3
pip install openfold3

# Run setup (downloads ~2GB model parameters + CCD data)
# When prompted, accept default cache directory (~/.openfold3)
# Choose "Download default model only"
# Skip integration tests
setup_openfold
```

### 3. Pre-Run the Examples

Run each demo prediction once before recording so MSAs are cached and you know the timing.

```bash
# Ubiquitin (small, fast — ~2-5 min on A100)
run_openfold predict \
    --query_json=examples/example_inference_inputs/query_ubiquitin.json \
    --output-dir ./demo_output/ubiquitin/

# Protein-ligand (Mcl-1 + ATP + drug — longer, ~5-10 min)
run_openfold predict \
    --query_json=examples/example_inference_inputs/query_protein_ligand.json \
    --output-dir ./demo_output/protein_ligand/
```

### 4. Install Visualization

On your local machine (not the GPU server):

```bash
# ChimeraX (free for academic use, best visuals)
# Download from: https://www.cgl.ucsf.edu/chimerax/download.html

# OR PyMOL (open-source version)
pip install pymol-open-source
```

### 5. Download Reference Structures

For the comparison overlay:

```bash
# Ubiquitin experimental structure
# Download 1UBQ from PDB: https://www.rcsb.org/structure/1UBQ
# Or via command line:
curl -o 1ubq.cif "https://files.rcsb.org/download/1UBQ.cif"
```

### 6. Transfer Output Files

Copy the predicted `.cif` files from the GPU server to your local machine for visualization.

```bash
scp gpu-server:demo_output/ubiquitin/ubiquitin/seed_*/sample_0_model.cif ./local_demo/
scp gpu-server:demo_output/protein_ligand/*/seed_*/sample_0_model.cif ./local_demo/
```

---

## On-Camera Demo: Scene by Scene

### Scene 1: "Three Commands" (60 seconds)

**What to show:** Terminal, large font (18pt+), dark background.

**What to say:** "Let me show you how easy this is. Three commands."

Type each command slowly and clearly. You can use pre-recorded output if the actual run is too slow.

```bash
# Command 1 — show typing this
pip install openfold3
```

*Wait for output, or cut to completion*

```bash
# Command 2 — show typing this
setup_openfold
```

*Show the interactive prompts briefly, then cut to completion*

```bash
# Command 3 — show typing this
run_openfold predict --query_json=examples/example_inference_inputs/query_ubiquitin.json
```

*Show the prediction starting, then cut to completion. Show the final output lines.*

**Key moment:** When the terminal shows prediction complete and lists output files.

---

### Scene 2: "What Does the Input Look Like?" (45 seconds)

**What to show:** The JSON input file in a text editor with syntax highlighting.

```bash
# Open in editor on camera
cat examples/example_inference_inputs/query_ubiquitin.json
```

**What to say:** "Here's what an input looks like. It's just JSON. You specify the molecule type — protein, DNA, RNA, or ligand — and give it the sequence."

Then show the protein-ligand example:

```bash
cat examples/example_inference_inputs/query_protein_ligand.json
```

**What to say:** "For a drug molecule, you give it a SMILES string — that's a text representation of a chemical structure. This one is predicting how a drug binds to the cancer target Mcl-1."

**Highlight on screen:** The `"smiles"` field and the `"molecule_type": "ligand"` field.

---

### Scene 3: "What Comes Out" (30 seconds)

**What to show:** Terminal listing the output directory.

```bash
ls -la demo_output/ubiquitin/ubiquitin/seed_1/
```

Output will show something like:
```
sample_0_model.cif
sample_0_confidences.json
sample_0_confidences_aggregated.json
sample_1_model.cif
...
timing.json
```

**What to say:** "For each seed, you get 5 candidate structures ranked by confidence, plus detailed confidence scores for every atom."

Optionally show the aggregated confidence JSON:

```bash
cat demo_output/ubiquitin/ubiquitin/seed_1/sample_0_confidences_aggregated.json
```

**Highlight:** `avg_plddt`, `ptm`, `sample_ranking_score` values.

---

### Scene 4: "Visualizing the Prediction" (90 seconds)

**What to show:** ChimeraX or PyMOL with the predicted structure.

#### ChimeraX Commands

```
# Open predicted structure
open local_demo/sample_0_model.cif

# Color by confidence (pLDDT is in B-factor)
color bfactor palette alphafold

# Nice view
lighting soft
set bgColor white

# Rotate slowly for B-roll
turn y 1 360
```

**What to say:** "Here's the predicted structure in ChimeraX, colored by confidence. Blue means the model is very confident — pLDDT above 0.9. Orange and red mean it's uncertain, usually flexible loops or disordered regions."

#### PyMOL Commands (alternative)

```
# Open predicted structure
load local_demo/sample_0_model.cif, predicted

# Color by B-factor (pLDDT)
spectrum b, red_white_blue, minimum=0, maximum=1

# Nice cartoon view
show cartoon
hide lines
set ray_shadow, 0
bg_color white

# Rotate for B-roll
mset 1 x360
util.mroll 1, 360, 1
mplay
```

---

### Scene 5: "Comparison to Experiment" (60 seconds)

**What to show:** Predicted structure overlaid on experimental structure.

#### ChimeraX Commands

```
# Open both structures
open local_demo/sample_0_model.cif
open 1ubq.cif

# Align them
matchmaker #2 to #1

# Color predicted = blue, experimental = orange
color #1 cornflowerblue
color #2 orange

# Show side by side, then overlay
```

#### PyMOL Commands (alternative)

```
load local_demo/sample_0_model.cif, predicted
load 1ubq.cif, experimental

# Align
align predicted, experimental

# Color
color marine, predicted
color orange, experimental

# Show both as cartoon
show cartoon
hide lines
bg_color white
```

**What to say:** "Blue is the prediction, orange is the experimental crystal structure. They're nearly identical. This is a protein that took crystallographers months to solve — the model does it in minutes."

**Key moment:** The overlay. Pause and let it sink in.

---

### Scene 6: "Drug Binding" (60 seconds)

**What to show:** The protein-ligand prediction in ChimeraX/PyMOL.

#### ChimeraX Commands

```
# Open protein-ligand prediction
open local_demo/protein_ligand_sample_0_model.cif

# Show protein as surface, ligand as sticks
surface #1
transparency #1 60
show #1/ligand sticks

# Color surface by hydrophobicity or electrostatic potential
color byattr  bfactor palette alphafold
```

#### PyMOL Commands (alternative)

```
load local_demo/protein_ligand_sample_0_model.cif, complex

# Show protein as surface
show surface, polymer
set transparency, 0.6

# Show ligand as sticks
show sticks, organic
color yellow, organic

bg_color white
```

**What to say:** "Here's a protein-ligand prediction. The drug molecule — shown in yellow — sits right in the protein's binding pocket. This is what drug discovery looks like in 2026. You type in a protein sequence and a chemical formula, and the model predicts exactly how they fit together."

---

## Timing Estimates

| Scene | Duration | Notes |
|-------|----------|-------|
| Three commands | 60s | Can cut between commands |
| Input files | 45s | Static screen, voiceover |
| Output files | 30s | Terminal listing |
| Visualization | 90s | Slow rotations for impact |
| Comparison overlay | 60s | The "wow" moment |
| Drug binding | 60s | The "this is the future" moment |
| **Total demo** | **~6 min** | Can trim to 4 min if needed |

---

## B-Roll to Capture During Demo

While you have the GPU and visualizations open, record extra footage:

- [ ] Slow protein rotation (30s loop, white background)
- [ ] Slow protein rotation (30s loop, black background)
- [ ] Close-up of ligand in binding pocket rotating
- [ ] Terminal scrolling during prediction (the "working" shots)
- [ ] Side-by-side predicted vs experimental from multiple angles
- [ ] Confidence coloring — zoom into a high-confidence region, then a low-confidence loop
- [ ] The JSON input file scrolling
- [ ] `setup_openfold` running (download progress bars)

---

## Fallback: If You Can't Get a GPU

Use NVIDIA NIM or Neurosnap for the prediction, and pre-download example output files for visualization. The visualization part (ChimeraX/PyMOL) runs on any machine — you only need the GPU for the actual prediction step.

You can also screen-record the Chai Discovery web interface (https://lab.chaidiscovery.com/) as a "hosted alternative" demo.
