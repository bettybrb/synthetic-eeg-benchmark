# Reproducibility

This document records the software environments, external repositories and compatibility changes used for the benchmark.

## Python environments

Both environments used **Python 3.11.7**. Separate environments were maintained because the legacy EEG classifier and VAE implementation depend on different software stacks.

### Classifier environment

From the repository root:

```bash
python3.11 -m venv env
env/bin/pip install -r requirements/classifier.txt
```

The classifier uses a pinned legacy Braindecode checkout:

```bash
git clone https://github.com/TNTLFreiburg/braindecode.git braindecode-legacy
git -C braindecode-legacy checkout d9feb5c6cfcd203fa8daa79ccd3217712714f330
env/bin/pip install -e ./braindecode-legacy
```

### VAE environment

```bash
python3.11 -m venv vae-env
vae-env/bin/pip install -r requirements/vae.txt
```

The hierarchical VAE implementation is pinned separately:

```bash
mkdir -p eeg_pipeline/external
git clone --branch hvEEGNet_paper https://github.com/jesus-333/Variational-Autoencoder-for-EEG-analysis.git eeg_pipeline/external/vae_repo
git -C eeg_pipeline/external/vae_repo checkout 010426ea09f4151adc91ee7fcf3e81a3280c51bf
```

## External repositories

### Braindecode

- Repository: `TNTLFreiburg/braindecode`
- Commit: `d9feb5c6cfcd203fa8daa79ccd3217712714f330`
- Expected local directory: `braindecode-legacy/`

### hvEEGNet / hierarchical VAE

- Repository: `jesus-333/Variational-Autoencoder-for-EEG-analysis`
- Branch: `hvEEGNet_paper`
- Commit: `010426ea09f4151adc91ee7fcf3e81a3280c51bf`
- Expected local directory: `eeg_pipeline/external/vae_repo/`

## Compatibility changes used in the experiment

The external VAE code required three compatibility changes in the experimental environment:

1. `library/dataset/download.py` — updated the MOABB dataset identifier from `BNCI2014001` to `BNCI2014_001`.
2. `library/model/hierarchical_VAE.py` — corrected the hierarchy-level activity check and added latent-shape validation.
3. `library/training/soft_dtw_cuda.py` — replaced nested `min` / `max` operations for Numba/CUDA compatibility.

The historical Braindecode environment also contained a local compatibility edit to `braindecode/datasets/bcic_iv_2a.py`. That edit was not preserved as a standalone patch, so exact byte-for-byte recreation of the original environment is not guaranteed. The benchmark protocol, dependency version and commit used for the experiments are preserved here.

## Data layout

The pipeline expects the BCI Competition IV Dataset 2a recordings under:

```text
eeg_pipeline/data/raw_data/
```

Processed splits and generated EEG are written under:

```text
eeg_pipeline/data/processed/
eeg_pipeline/data/generated/
```

These directories are excluded from Git.

## Experimental configuration

The final configuration is defined in `eeg_pipeline/pipeline/config.py`.

Key protocol settings:

- participants: 1–9;
- classifier seeds: 0, 1, 2;
- generator seeds: 0, 1, 2;
- development split seed: 42;
- 259 training trials and 29 validation trials per participant;
- 288 trials from the separate evaluation session for final testing;
- 22 EEG channels;
- 1000 samples per trial;
- ShallowFBCSPNet classifier;
- maximum 120 classifier epochs with validation-based early stopping.

The final runner verifies the expected experiment combinations and requires **999 successful primary benchmark rows** before marking the run complete.

## Running the benchmark

After installing both environments, preparing the external repositories and placing the EEG data in the expected directory, run from the repository root:

```bash
bash scripts/run_final_pipeline.sh
```

The runner exports the real participant splits, trains the neural generators, validates generated arrays, runs the downstream classifier benchmark and creates result summaries.

The H-VAE path requires a CUDA-capable environment compatible with the pinned dependencies.

## Stored results

Large run-level outputs, checkpoints and generated EEG arrays are not distributed in the repository. Compact method-level, participant-level and ratio summaries are retained under `results/`.
