# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

OpenFold3-preview is a biomolecular structure prediction model — a bitwise reproduction of DeepMind's AlphaFold3 by the AlQuraishi Lab (Columbia) and the OpenFold consortium. It predicts structures for proteins, RNA, DNA, and small molecules. Package version `0.4.0`, Apache 2.0, Python >= 3.10, Linux/CUDA only. Full docs at https://openfold-3.readthedocs.io.

## Commands

The package installs two console scripts (`pyproject.toml [project.scripts]`): `run_openfold` (→ `openfold3.run_openfold:cli`, a `click` group) and `setup_openfold`. CLI options accept either dashes or underscores (e.g. `--query-json` == `--query_json`).

```bash
# Install + download model params and CCD data (params cached to $OPENFOLD_CACHE, default ~/.openfold3/)
pip install openfold3        # use .[cuequivariance] for cuEq kernel acceleration (needs torch>=2.7)
setup_openfold

# Predict (only --query_json is required; MSA server + templates on by default)
run_openfold predict --query_json=examples/example_inference_inputs/query_ubiquitin.json --output_dir ./output
# Tune behavior with a runner yaml (see examples/example_runner_yamls/): low_mem.yml,
# multiple_gpu.yml, cuequivariance.yml, affinity.yaml, save_msa_output.yml
run_openfold predict --query_json=... --runner_yaml=examples/example_runner_yamls/low_mem.yml

# MSA alignment only (writes <output_dir>/query_msa.json)
run_openfold align-msa-server --query_json=... --output_dir ./output

# Train (requires a runner yaml; see examples/training_yamls/initial_training.yml)
run_openfold train --runner_yaml=examples/training_yamls/initial_training.yml --seed 42

# Lint
ruff check openfold3/

# Tests (testpaths=openfold3/tests is preconfigured, so bare `pytest` works)
pytest                                          # all tests
pytest openfold3/tests/test_pairformer.py       # single file
pytest -m inference_verification                # post-setup install check
pytest -m "not slow"                            # skip large/slow tests
pytest -n auto                                  # parallel (pytest-xdist)
```

## Inference query format

Inference input is a JSON list of queries validated by the Pydantic `InferenceQuerySet` (`openfold3/projects/of3_all_atom/config/inference_query_format.py`). See `examples/example_inference_inputs/` for the full range: single/multimer protein, homomer, protein+ligand, DNA with PTMs. This is the schema to read when changing what inputs the model accepts.

## Architecture

The codebase separates **reusable core** (`openfold3/core/`) from **project-specific assembly** (`openfold3/projects/of3_all_atom/`). The project layer wires core components into the concrete OF3 all-atom model and owns its configs; core stays model-agnostic.

**Entry / config flow.** `run_openfold.py` (CLI) builds a Pydantic config from `entry_points/validator.py` (`InferenceExperimentConfig` / `TrainingExperimentConfig`), then constructs an `ExperimentRunner` from `entry_points/experiment_runner.py`. The base `ExperimentRunner` has two subclasses: `TrainingExperimentRunner` (PyTorch Lightning) and `InferenceExperimentRunner`. Configuration is `ml_collections.ConfigDict` populated from YAML and overlaid with CLI flags and Pydantic validators. CLI flags > runner yaml > defaults.

**Project layer** (`projects/of3_all_atom/`): `model.py` (the `OpenFold3` `nn.Module`), `runner.py` (project runner), `project_entry.py`, and `config/` — `model_config.py`, `dataset_configs.py`, `features.py`, `inference_query_format.py`, plus preset bundles in `model_setting_presets.yml`.

**Core model** (`core/model/`): `feature_embedders/` (input embedding), `latent/` (`msa_module.py`, `pairformer.py`, `evoformer.py`, `template_module.py`, with `base_blocks.py`/`base_stacks.py`), `structure/` (`diffusion_module.py`), `heads/` (confidence/distogram outputs), `primitives/` (Linear, Attention), `layers/` (transformer primitives).

**Model execution** (`core/runners/`): `model_runner.py` drives the forward pass; `writer.py` serializes predictions.

**Data pipeline** (`core/data/`): `framework/` (loading), `io/` (parsing/serialization), `pipelines/` (MSA + template processing), `tools/` (e.g. `colabfold_msa_server.py`), `primitives/`, `resources/`. CCD ligand data and model params are fetched by `setup_openfold`.

**Loss / metrics** (`core/loss/`, `core/metrics/`): diffusion, confidence, distogram losses; quality + confidence metrics.

**Kernels** (`core/kernels/`): optional acceleration via cuEquivariance (`cueq_utils.py`) and Triton (`triton/`); DeepSpeed4Science EvoformerAttention configs in `deepspeed_configs/`. All optional — model runs without them but slower.

**Geometry** (`core/utils/geometry/`): Kabsch alignment, SE(3) rigid transforms, rotation matrices.

**Data-prep scripts** (`scripts/data_preprocessing/`): build training/validation dataset caches (PDB, RNA/protein monomers) and convert caches to LMDB — these are how training data is materialized, separate from the inference path.

## Lint & Style

Ruff, line length 88. Rules: E, F, UP, B, SIM, I. Ignored: E741, SIM108, B905; E501 ignored under `**/tests/**`. `UP006` is a safe autofix. Run `ruff check openfold3/` before committing.
