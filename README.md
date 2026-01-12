# EEG-Alpha-Analysis

Notebook-driven pipeline for **eyes-open (EO)** vs **eyes-closed (EC)** EEG classification, labeling, and downstream analyses (PSD/FOOOF, alpha power, plots).

## Quick Start (recommended run order)

1) **If you start from labeled EDF recordings (NEW dataset)**: run `converting_edf_to_fif.ipynb` → produces labeled epoch `.fif` files.  
2) **(Optional, speed-up for ONE_MAIN_FOOOF runs)**: run `Precompute_ONE_MAIN_FOOOF.ipynb` → precomputes caches used by FOOOF-based inference.  
3) **Train/evaluate**: run `EC_EO_Classifier.ipynb` → writes run artifacts under `outputs/`.  
4) **Label a large unlabeled `.set` dataset**: run `Label_with_EC_EO_Classifier.ipynb` → writes `label_predictions.csv` per trained run.  
5) **Analyze + visualize results**: run `Classifier_Results_Visualization.ipynb`, `analysis_script.ipynb`, and/or the `mean_alpha_power*.ipynb` notebooks.

## What to run for X

- **Train + evaluate an EC/EO model**: `EC_EO_Classifier.ipynb`
- **Label a large unlabeled dataset with a trained run**: `Label_with_EC_EO_Classifier.ipynb`
- **Make figures / analyze results**: `Classifier_Results_Visualization.ipynb` (general), `analysis_script.ipynb` (compact end-to-end), `mean_alpha_power.ipynb` / `mean_alpha_power_EC_EO.ipynb` (alpha power analyses)

## Key Conventions (keep in mind)

### Datasets and labels
- **Old labeled dataset**: EEGLAB epoch `.set` files split into EO vs EC files; labeled epochs are selected via `reject.rejmanual`.
- **New labeled dataset**: MNE epoch `.fif` files (typically `sub####a_epo.fif` and `sub####b_epo.fif`) with event labels `EO`, `EC`, `OTHER`.
- **Big unlabeled dataset**: many preprocessed EEGLAB `.set` files labeled by running inference (`Label_with_EC_EO_Classifier.ipynb`).
- Label convention used throughout: **EO = 0**, **EC = 1**, **OTHER = ignored**.
- For new data, training uses an **A/B union** (fill missing EO/EC labels) and **drops a/b conflicts**, logging them to CSV.

### Preprocessing and channels
- New EDF→FIF conversion uses the same core preprocessing choices: **200 Hz**, **1–70 Hz band-pass**, **50 Hz notch**, and **1 s epochs**.
- Channel naming is normalized (e.g., stripping `EEG ` and `-REF`), and most workflows assume a consistent **19-channel** 10–20 montage/order.

### Configuration (paths via env vars)
- Old labeled `.set` paths: `EC_EO_OPEN_DIR`, `EC_EO_CLOSED_DIR`
- New labeled `.fif` path: `NEW_EEG_PROCESSED_DIR`
- Big unlabeled `.set` path: `PREPROCESSED_SETFILES_DIR`
- Labeling outputs/caches: `LABELING_DIR`, `SAVED_FOOOF_DIR`

## Notebooks

### `data_new.ipynb` (data visualization; NEW dataset)
Data visualization notebook for inspecting spectral/alpha characteristics in the **new** dataset (PSD plots, alpha power plots, scalp topographies, rejected epochs, and channel-related summaries).  
Note: this notebook is a **modified version** of a file from **Rand & Jessen (2025)** and has been modified to fit this project.

### `data_old.ipynb` (data visualization; OLD dataset)
Data visualization notebook for inspecting spectral/alpha characteristics in the **old** dataset (PSD plots, alpha power plots, scalp topographies, rejected epochs, and channel-related summaries).  
Note: this notebook is a **modified version** of a file from **Rand & Jessen (2025)** and has been modified to fit this project.

### `converting_edf_to_fif.ipynb` (EDF → labeled epoch FIF)
Converts labeled **EDF** recordings into preprocessed **MNE `Epochs` `.fif`** files (fixed-length epochs, standardized channel set/order, and per-epoch labels derived from EDF annotations).  
Note: this notebook is a **modified version** of a file from **Rand & Jessen (2025)** and has been modified to fit this project.

### `Precompute_ONE_MAIN_FOOOF.ipynb` (FOOOF cache precompute)
Precomputes caches used by **ONE_MAIN_FOOOF** to avoid re-fitting spectral models repeatedly. Depending on configuration, it can cache:
- Per-subject alpha profile parameters (e.g., center frequency/bandwidth), and/or
- Full per-epoch precomputed features for faster inference.

### `EC_EO_Classifier.ipynb` (main training + evaluation)
The main end-to-end notebook to:
- Resolve clinical data inputs (old `.set` and optionally new `.fif` epochs)
- Extract features (PSD or FOOOF/specparam; optional `ONE_MAIN_FOOOF`)
- Train/evaluate **logistic regression** with subject-wise CV/holdout and optional temporal smoothing
- Save artifacts for reuse (models, scalers/imputers, predictions tables, plots) under `outputs/<run_folder>/`

### `Classifier_Results_Visualization.ipynb` (compare runs, plots, stats)
Loads outputs from classifier runs and produces analysis/figures such as:
- Performance summaries (accuracy, confusion matrices, ROC curves)
- PSD vs FOOOF comparisons
- Additional descriptive analyses (e.g., alpha/FOOOF parameter trends)
- Statistical tests comparing model variants (where applicable)

### `Label_with_EC_EO_Classifier.ipynb` (label big unlabeled dataset)
Applies trained run artifacts from `outputs/<run_folder>/` to a large collection of unlabeled EEGLAB `.set` files and writes a per-epoch predictions table:
- Output: `label_predictions.csv` (one per run folder)
- Supports FOOOF runs by reusing precomputed caches when available.

### `mean_alpha_power.ipynb` (alpha power analysis; EC-focused selection)
Computes mean absolute/relative alpha power across age (and related plots), selecting EC epochs per subject using run-based logic (e.g., longest consecutive EC run and confidence fallbacks).  
Note: this notebook is a **modified version** of a file from **Rand & Jessen (2025)** and has been modified to fit this project.

### `mean_alpha_power_EC_EO.ipynb` (alpha power analysis; EC vs EO)
Alpha power analysis comparing **EC vs EO** by selecting **top-K epochs per subject per class** using classifier probabilities (e.g., highest `prob_ec` for EC and lowest `prob_ec` for EO), then computing and plotting mean absolute/relative alpha power.

### `analysis_script.ipynb` (end-to-end analysis script notebook)
Runs a compact end-to-end analysis on labeled/predicted data, including:
- FOOOF-derived center frequency (CF) extraction (where available) and CF vs age plots
- Epoch selection (e.g., top-K EC/EO)
- Mean relative alpha calculations and visualizations
- EC vs EO alpha power comparisons and summary outputs
