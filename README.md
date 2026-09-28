# Synthetic EEG Benchmark

### Evaluating the downstream utility of synthetic EEG for motor imagery classification

**MSc Artificial Intelligence research project — Queen Mary University of London**

This repository benchmarks whether synthetic EEG can replace or augment participant-specific real EEG for four-class motor imagery classification. Generators are fitted only on real training data, model selection uses real validation data, and final performance is measured on a separate real recording session.

![Results overview](assets/results_overview.png)

## Key results

| Training condition | Accuracy on real held-out EEG |
| --- | ---: |
| Real EEG baseline | **69.24% ± 16.06** |
| H-VAE reconstruction | **68.06% ± 16.62** |
| Best Gaussian generator | **31.91% ± 5.04** |
| Conditional VAE generation | **25.73% ± 1.57** |

*Values are macro means across nine participants; ± denotes participant-level standard deviation.*

The strongest result came from **H-VAE reconstruction**, which retained most of the downstream classification utility of the original trials. This is importantly different from independent generation: each reconstruction originates from an encoded real EEG trial.

Independent generation was substantially weaker. The best Gaussian method reached **31.91%**, while conditional VAE generation reached **25.73%**, close to the **25% chance level** for four classes.

Mixing real and Gaussian synthetic EEG also did not improve on real-only training. Replacing 50% of the real training set reduced accuracy to **56.35%**. Adding synthetic EEG equal to 50% of the real training-set size reached **62.12%**, compared with **69.24%** for real-only training.

## Benchmark at a glance

| | |
| --- | --- |
| Dataset | BCI Competition IV Dataset 2a |
| Participants | 9 |
| Motor imagery classes | Left hand, right hand, feet, tongue |
| EEG channels | 22 |
| Sampling rate | 250 Hz |
| Trial representation | 22 × 1000 samples (4 seconds) |
| Development split | 259 train / 29 validation trials per participant |
| Final test set | 288 real trials from a separate recording session |
| Downstream classifier | ShallowFBCSPNet |
| Classifier seeds | 3 |
| Generator seeds | 3 |
| Primary benchmark runs | **999 successful classifier runs** |

## Research question

**Can synthetic EEG replace participant-specific real EEG for classifier training without losing performance on a separate real recording session, and can synthetic data help through partial replacement or augmentation?**

The benchmark was designed around downstream utility rather than visual similarity: synthetic EEG is useful only if a classifier trained on it can generalise to real unseen EEG.

## Experimental protocol

```text
BCI Competition IV 2a — repeated independently for each participant

Session 1: development data (288 real trials)
├── 259 training trials ──> fit generator ──> real / synthetic / mixed classifier training
└──  29 validation trials ─────────────────> model selection

Session 2: 288 real trials ────────────────> final held-out evaluation
```

The generator never receives validation or test EEG. Validation and testing remain real in every condition.

## Methods compared

| Method family | Role in the benchmark |
| --- | --- |
| Real EEG | Reference baseline |
| 8 Gaussian generators | Statistical controls conditioned on combinations of class, channel and time |
| H-VAE / hvEEGNet | Reconstruction-based synthetic EEG |
| Conditional VAE | Independent class-conditioned generation from a latent prior |
| Class-specific VAE | Exploratory class-specific generative variant |
| Hierarchical conditional VAE | Exploratory hierarchical latent-prior variant |
| Replacement experiments | Replace increasing fractions of real training EEG with synthetic EEG |
| Augmentation experiments | Add increasing amounts of synthetic EEG to the complete real training set |

All conditions use the same downstream classifier so that the main experimental variable is the source of the classifier training data.

## Main contribution

The project provides one controlled evaluation framework connecting multiple synthetic EEG approaches to the same participant-specific data splits, classifier, real validation protocol and held-out real test session.

The implementation includes:

- a unified real-versus-synthetic classification protocol across all nine participants;
- eight Gaussian controls for testing which marginal statistics are useful;
- H-VAE reconstruction using hvEEGNet and Soft-DTW;
- conditional, class-specific and hierarchical VAE generation;
- independent classifier and generator seeds;
- replacement and augmentation ratio experiments;
- automatic completeness checks for all **999 primary benchmark runs**;
- participant-level and method-level result summaries.

## Interpretation

The results separate **signal reconstruction** from **independent synthetic generation**.

H-VAE reconstruction achieved performance close to the real-data baseline because it preserved information already present in encoded real trials. It should therefore not be interpreted as evidence that the model learned to generate equally useful EEG independently.

By contrast, samples generated independently from Gaussian or learned VAE priors did not preserve enough class-discriminative structure to replace participant-specific real EEG under this protocol. Increasing Gaussian replacement consistently reduced performance, while Gaussian augmentation also remained below the real-only baseline.

## Repository structure

```text
synthetic-eeg-benchmark/
├── eeg_pipeline/
│   ├── pipeline/          Core data, classifier, generator and evaluation logic
│   └── experiments/       Generation and benchmark experiment entry points
├── scripts/
│   └── run_final_pipeline.sh
├── results/
│   ├── method_summary.csv
│   ├── participant_summary.csv
│   └── ratios/
├── assets/
│   └── results_overview.png
├── docs/
│   ├── reproducibility.md
│   └── upstream-high-gamma-license.txt
└── requirements/
    ├── classifier.txt
    └── vae.txt
```

## Results and reproducibility

The repository keeps compact final summaries rather than thousands of intermediate run rows:

- [`results/method_summary.csv`](results/method_summary.csv) — primary method comparison;
- [`results/participant_summary.csv`](results/participant_summary.csv) — participant-level results;
- [`results/ratios/`](results/ratios/) — replacement and augmentation summaries.

Raw EEG recordings, generated EEG arrays, trained model checkpoints and logs are intentionally not stored in Git.

Exact environment versions, external repository commits, compatibility modifications and execution instructions are documented in [`docs/reproducibility.md`](docs/reproducibility.md).

## Limitations

The benchmark contains nine participants from one motor imagery dataset and evaluates participant-specific models. Results therefore describe this experimental protocol rather than synthetic EEG in general. Reconstruction and independent generation are also fundamentally different tasks and are reported separately for that reason.

## Tech

**Python · PyTorch · Braindecode · NumPy · Numba/CUDA · EEG · Brain-computer interfaces · Variational autoencoders · Synthetic data · Experimental evaluation**

## Upstream components

The project uses pinned versions of legacy Braindecode and an external hvEEGNet implementation. Their exact commits and the compatibility changes used for this benchmark are documented in the reproducibility notes. An upstream license notice retained from the legacy High-Gamma/Braindecode codebase is stored separately under `docs/`.
