# Multimodal Free-Space Photonic Extreme Learning Machine (PELM)

This repository contains the code used to run and analyze a free-space **Photonic Extreme Learning Machine (PELM)** across four data modalities — **MNIST** (image), **FSDD** (audio, log-Mel spectrograms), **Mushroom** (binary tabular), and **Abalone** (tabular regression) — using a phase-only SLM, free-space (Fourier-like) propagation, and camera intensity readout, with a digitally trained ridge regression readout.

The architecture and results are described in detail in the accompanying paper:

 **Paper (PDF):** [Multimodal Optical Feature Extraction with a Free-Space PELM](https://arxiv.org/pdf/2605.29043)

 **Pre-computed feature matrices (`.npz`):** [NPZ files - Google Drive](https://drive.google.com/drive/folders/15WQ4clja7MEvxCUGoy_Oq0b7v5QBxdXE)

For the physical setup, encoding equations, embedding definitions, and the full set of empirical diagnostics (distance preservation, CKA, kernel fits, t-SNE), please refer to the paper above — this README focuses on **how to run the code**.

---

## Repository layout

```
.
├── dlls/                       # Required DLLs for the Thorlabs / Holoeye SDKs (Windows)
├── thorlabs_tsi_sdk/            # Thorlabs TSI camera SDK (Windows-only)
├── camera.py                   # Thorlabs camera wrapper used by optics_driver.py
├── windows_setup.py             # One-time Windows environment / SDK path setup script
├── .gitignore
├── README.md
│
└── pelm/                        # Main experiment code (run everything from inside here)
    ├── config.py                # Master configuration (dataset, sizes, embedding, hyperparams)
    ├── main.py                  # Main experiment runner (hardware acquisition + training/testing)
    ├── pelm_core.py              # PELM algorithm: encoding, embedding mask, feature extraction, readout
    ├── optics_driver.py          # Hardware driver: SLM display + camera capture (OpticalSystem class)
    ├── lambda_cv.py               # 5-fold CV lambda sweep + final test evaluation from saved .npz files
    ├── kernel.py                  # Empirical optical kernel vs. theoretical kernel comparison
    ├── analysis_2.py               # CKA, linear vs. RBF, accuracy plots, kernel heatmaps, t-SNE
    ├── isometry.py                 # Distance-preservation (isometry) plots across datasets
    ├── requirements.txt
    │
    ├── data_loader/                # Dataset loaders (MNIST, FSDD, Abalone, Mushroom, etc.)
    ├── data/                        # Raw/downloaded datasets (auto-downloaded where applicable)
    ├── HEDS/                        # Holoeye SLM SDK (Windows-only, place here yourself)
    │
    ├── npz_files/                   # Saved optical feature matrices (H_train / H_test per dataset+embedding)
    ├── results/                     # Outputs from lambda_cv.py (CV curves, confusion matrices, summaries)
    ├── kernel_analysis/              # Outputs from kernel.py, analysis_2.py, isometry.py
    ├── analysis_results/             # Additional analysis outputs
   
```

> `dlls/`, `thorlabs_tsi_sdk/`, `camera.py`, and `windows_setup.py` at the repo root provide the Windows hardware SDK dependencies used by `pelm/optics_driver.py`. Run `windows_setup.py` once on the acquisition machine to set up SDK paths/DLLs before running `pelm/main.py`.

---

## Setup

1. Create a virtual environment and install the requirements:
   ```bash
   cd pelm
   pip install -r requirements.txt
   ```
2. Hardware SDKs (Windows-only acquisition machine):
   - Run `windows_setup.py` (repo root) once to set up SDK paths and copy the required DLLs from `dlls/`.
   - The Thorlabs camera SDK lives in `thorlabs_tsi_sdk/`, and `camera.py` (repo root) wraps it for use by `pelm/optics_driver.py`.
   - The Holoeye SLM SDK (HEDS) is **not included** — place it inside `pelm/HEDS/` yourself.

---

## Step 1 — Optical system setup and check

Before running any experiment, align and verify the optical setup (laser, SLM, 4f propagation path, iris/first-order selection, polarizer, camera). `optics_driver.py` (the `OpticalSystem` class) talks directly to the SLM and camera and includes a built-in `run_optical_test()` sanity check that is called automatically at the start of `main.py`.

**For the full optical schematic, alignment notes, and hardware parameters (laser wavelength, SLM/camera specs, settling times, etc.), see supplymentry material attached in paper.**

---

## Step 2 — Training and testing (data acquisition)

1. Open `config.py` and set it up for your run:
   - `DATASET_TYPE`: `"mnist"`, `"fsdd"`, `"mushroom"`, or `"abalone"`
   - `ENCODING_METHOD`: `"noise_embedding"` or `"fourier_embedding"` (DRF embedding is handled separately, see `kernel.py`/`analysis_2.py` toggles)
   - `N_TRAIN`, `N_TEST`: dataset sizes (uncomment/edit the relevant block — quick test vs. full experiment)
   - `M_FEATURES`, `LAMBDA_REG`, `CAM_EXPOSURE_US`, and other hardware/embedding parameters as needed

2. Run the main experiment:
   ```bash
   python main.py
   ```
   - **`main.py`** is the runner you actually execute. It loads the chosen dataset, initializes `OpticalSystem` (from `optics_driver.py`) and `PELM_Algorithm` (from `pelm_core.py`), runs the optical test, then loops over all samples: encodes each input as a phase mask, displays it on the SLM, captures the camera frame, and extracts the `M`-dimensional feature vector.
   - **`pelm_core.py`** contains the core PELM logic — generating/loading the fixed embedding mask `W`, encoding inputs into phase patterns, extracting binned camera features, and the ridge-regression readout.
   - **`optics_driver.py`** initializes and controls the actual hardware (SLM display + camera streaming/capture). This must run on the machine connected to the optical setup.
   - Training automatically checkpoints every `CKPT_INT` samples (set per dataset in `config.py`) and can be resumed if interrupted.
   - After training, a ridge-regression readout (`beta`) is fit and the model is evaluated on the test set; accuracy/RMSE, confusion matrices, and `H_matrix.png` are saved (please note to run lambda_cv.py once .npz files are obtained as it contains preprocessing steps mentioned).

---

## Step 3 — Save the feature matrices (`.npz`)

During/after the run, `main.py` saves the measured optical feature matrices as:

```
<dataset>_train_fold1.npz   # H_train, y_train, last_idx
<dataset>_test_fold1.npz    # H_test,  y_test,  last_idx
```

These are later renamed/organized into `npz_files/<dataset>_train_fold1_<embedding>.npz` and `npz_files/<dataset>_test_fold1_<embedding>.npz` (where `<embedding>` is `noise`, `fourier`, or `drf`) — this naming convention is what all downstream analysis scripts (`lambda_cv.py`, `kernel.py`, `analysis_2.py`, `isometry.py`) expect.

---

## Step 4 — Lambda cross-validation and test evaluation

If you don't want to run the optical hardware yourself, you can use the **pre-computed `.npz` feature files** provided here:

[NPZ files — Google Drive](https://drive.google.com/drive/folders/15WQ4clja7MEvxCUGoy_Oq0b7v5QBxdXE)

Place these into `npz_files/`, then run:

```bash
python lambda_cv.py
```

- Set `TARGET_DATASET` at the top of the script (`"mnist"`, `"fsdd"`, `"mushroom"`, `"abalone"`).
- For each embedding in `EMBEDDINGS = ["drf", "noise", "fourier"]` (run in parallel), this script:
  1. Performs 5-fold cross-validation over a log-spaced grid of `λ` (`LAMBDA_GRID`) to pick the best regularization strength,
  2. Retrains the readout on the full training set with that `λ`,
  3. Evaluates once on the held-out test set,
  4. Saves the CV curve, confusion matrix (classification) or NRMSE summary (Abalone), and the trained `beta` to `results/<dataset>/<embedding>/`.

---

## Step 5 — Feature-space diagnostics (kernel, CKA, isometry)

These scripts use the same `npz_files/<dataset>_*_fold1_<embedding>.npz` files to reproduce the empirical diagnostics discussed in the paper (Section 5):

- **`kernel.py`** — Computes the empirical optical kernel from the measured features and compares it against the theoretical kernels (`K_phase`, `K_Gaussian`, arccosine `K1`/`K2`, angular RBF), binned by input angle θ. Outputs go to `kernel_analysis/`.

- **`analysis_2.py`** — Computes Centered Kernel Alignment (CKA, linear vs. RBF), per-class separation (Cohen's d) vs. accuracy plots, empirical kernel heatmaps, and t-SNE visualizations of the optical feature space.

- **`isometry.py`** — Computes pairwise distance preservation (Pearson/Spearman correlation between input-space and optical-feature-space distances) and produces the 2×2 isometry grid (per-dataset + combined) for a chosen embedding.

**Toggling datasets / embeddings:** each script has a small configuration block near the top:
- `TARGET_DATASET` (or `TARGET_EMBEDDING` in `isometry.py`) selects which dataset/embedding to analyze.
- `EMBEDDINGS = ["drf", "noise", "fourier"]` controls which embeddings are compared.

Edit these toggles and re-run the script to reproduce diagnostics for a different modality or embedding.

---

## Notes

- Random seeds are fixed (`RANDOM_SEED = 42` in `config.py`) so that the embedding mask `W`, dataset splits, and cross-validation folds are reproducible.
- For details on the underlying physics (Eqs. 1–13), dataset descriptions, and full results tables, see the paper PDF linked at the top of this README.
