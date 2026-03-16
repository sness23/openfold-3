# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

OpenFold3-preview is a biomolecular structure prediction model — a bitwise reproduction of AlphaFold3 by the AlQuraishi Lab (Columbia) and the OpenFold consortium. It predicts structures for proteins, RNA, DNA, and small molecules. Licensed Apache 2.0, requires Python >= 3.10.

## Commands

```bash
# Install
pip install openfold3

# Setup (downloads model parameters and CCD data)
setup_openfold

# Predict structures
run_openfold predict --input queries.json --output_dir ./output

# Train
run_openfold train --config config.yaml

# MSA alignment only
run_openfold align-msa-server --input queries.json --output_dir ./output

# Lint
ruff check openfold3/

# Run all tests
pytest openfold3/tests/

# Run a single test file
pytest openfold3/tests/test_utils.py

# Run tests with coverage
pytest openfold3/tests/ --cov=openfold3

# Run only inference verification tests (post-setup check)
pytest openfold3/tests/ -m inference_verification
```

## Architecture

**Entry points** (`openfold3/run_openfold.py`, `openfold3/setup_openfold.py`): CLI with `predict`, `train`, `align-msa-server` commands. Setup script handles parameter downloads and CCD data from S3.

**Experiment runners** (`openfold3/entry_points/experiment_runner.py`): Abstract `ExperimentRunner` base with `TrainingExperimentRunner` (PyTorch Lightning) and `InferenceExperimentRunner` subclasses. Configuration uses `ml_collections.ConfigDict` with Pydantic validators.

**Project implementation** (`openfold3/projects/of3_all_atom/`): Contains the main `OpenFold3` model class (`model.py`), project-specific runner (`runner.py`), and config files for model architecture and dataset specifications.

**Core model** (`openfold3/core/model/`): Neural network architecture split into `feature_embedders/` (input processing), `latent/` (MSA module, Pairformer), `layers/` (transformer primitives), `heads/` (output predictions), `structure/` (diffusion module), and `primitives/` (Linear, Attention).

**Data pipeline** (`openfold3/core/data/`): Framework for data loading (`framework/`), I/O handling (`io/`), MSA and template processing (`pipelines/`), and tool interfaces (`tools/` — ColabFold server, parsers).

**Loss & metrics** (`openfold3/core/loss/`, `openfold3/core/metrics/`): Diffusion, confidence, and distogram losses; quality and confidence metrics.

**Kernels** (`openfold3/core/kernels/`): Optimized CUDA kernels with optional cuEquivariance and DeepSpeed4Science acceleration.

**Geometry** (`openfold3/core/utils/geometry/`): 3D geometry ops — Kabsch alignment, SE(3) rigid transformations, rotation matrices.

## Lint & Style

Ruff with line length 88. Rules: E, F, UP, B, SIM, I. Ignored: E741, SIM108, B905. E501 (line length) is ignored in tests.
