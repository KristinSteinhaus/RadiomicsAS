# RadiomicsAS

# CMR-Radiomics for Myocardial Fibrosis in Aortic Stenosis

Quantitative CMR radiomics to identify **biopsy-proven myocardial fibrosis** in severe aortic stenosis (AS) patients undergoing TAVR.  
This repository contains the pipeline for feature extraction, reproducibility filtering (ICC), unsupervised representations 
(PCA / consensus-clustering medoids), and supervised models with cross-validation. 

---

## Repository layout

```
radiomics/
├─ data/
│  ├─ raw/                # (ignored) original .mat / DICOM-derived inputs
│  ├─ processed/          # processed features, merged labels, etc.
│  ├─ clustering/         # clustering diagnostics
│  ├─ icc/                # inter/intraobserver reproducibility
│  └─ pca/                # PCA components, loadings, proxies
├─ notebooks/
│  ├─ 00_basic/           # exploratory checks
│  ├─ 01_reproducibility/ # ICC & mosaics
│  └─ 02_ml/              # ML experiments & figures
├─ reports/
│  └─ figures/            # exported plots (basics, clustering, features, pca, reproducibility, roc)
├─ src/
│  ├─ control/            # viewers/utilities
│  ├─ ml/                 # modeling helpers, clustering, plots
│  ├─ preprocessing/      # feature extraction, normalization, ROI tools
│  ├─ reproducibility/    # ICC, DeLong comparisons
│  ├─ stats/              # baseline tables, stats helpers
│  ├─ utils/              # cleaning, formatting, metrics
│  └─ viz/                # plotting style & paths
├─ requirements.txt
├─ README.md
└─ .gitignore
```

> `data/raw/` is intentionally **untracked**. To keep the folder visible, add a placeholder `data/raw/.gitkeep` and ensure `.gitignore` contains:
>
> ```
> /data/raw/*
> !/data/raw/.gitkeep
> ```

---

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scriptsctivate
pip install -r requirements.txt
export PYTHONPATH="${PWD}"     # PowerShell: $env:PYTHONPATH=(Get-Location)
```

## Data expectations

- Per patient/sequence `.mat` in `data/raw/<sequence>/ASxxx.mat`
  - `image_n_mask`: 2 channels → `[...,0]=image`, `[...,1]=mask`
  - `px_size`: `[sx, sy, (optional sz)]` in mm
- Labels (Excel) with `AS_ID`, `totale_fibrose`  
  → Code creates `Histopathology_label = (totale_fibrose > 0.11)`.

---

## Feature extraction (CLI)

From repo root:
```bash
source .venv/bin/activate
export PYTHONPATH="${PWD}"
python -m src.preprocessing.compute_radiomic_features
```

## Reproducibility & filtering

- Inter-/intraobserver ICC in `data/icc/`.
- Downstream models use only features with **ICC ≥ 0.80** (intersection of intra/inter).

---

## Unsupervised representations

- **PCA** — 5 PCs capture most variance; single PC-aligned proxies available.
- **Consensus clustering** — choose \(k\) via CDF / delta-area; use **medoids** as cluster representatives  
  (cine typically **k=8**; compact comparison **k=5**).

Artifacts in `data/pca/`, `data/clustering/`; figures in `reports/figures/`.

---

## Supervised models

- Algorithms: Logistic Regression, SVM, Naive Bayes, Random Forest, Gradient Boosting
- Validation: **repeated stratified 5-fold CV**
- Probabilities via `predict_proba` → **ROC–AUC** (primary metric)
- Example RF: `class_weight='balanced'`, `n_estimators=10` (others default)

---

## Reproducing figures

Use notebooks under `notebooks/`:
- `00_basic/*` — exploratory plots
- `01_reproducibility/*` — ICC & mosaics
- `02_ml/*` — ROC/AUC, PCA/medoid panels

Exports are written to `reports/figures/`.

---

## Paths & configuration

- Paths are centralized in `src/viz/paths.py` and `src/utils/`.
- Adjust there if your local data directories differ.

---

## Git hygiene

Ignore raw data and keep a visible (empty) folder:
```gitignore
/data/raw/*
!/data/raw/.gitkeep
```

If raw data were already committed, remove from tracking (files remain locally):
```bash
git rm -r --cached data/raw
git commit -m "Stop tracking data/raw"
```

---

## Citation

Please cite the master’s thesis and reference this repository when using the code or results.

---

## License

All rights reserved (internal use only).

---

## Contact

Maintainer: **Kristin Steinhaus** — <kristin.steinhaus@med.uni-goettingen.de>  
Supervisor: **Ute von Jan (PLRI)**, **Anne-Christin Hauschild (JLU)**
