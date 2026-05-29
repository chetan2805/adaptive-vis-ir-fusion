Reproducibility package — Adaptive Visible–Infrared Image Fusion
Code and instructions accompanying:
> C. N. Madivalar, "Adaptive Visible–Infrared Image Fusion with Saliency-Driven
> Band Weighting: A Saliency-versus-Precision Weight-Inversion Analysis for
> Dynamic Environmental and Industrial Imaging."
This package reproduces the empirical evaluation in Section 10 and provides the
additional baselines and metrics needed to strengthen the study for a Q1
submission. Everything here is training-free and runs on CPU.
> **Before submission:** replace the repository URL and Zenodo DOI placeholders
> in the manuscript's Data Availability statement with the real links you obtain
> after depositing this package (see "Archiving" below).
---
What is implemented
Fusion methods (`src/fusion_methods.py`)
`fixed_weight` — 0.5·IR + 0.5·VIS (canonical baseline)
`pca` — first-eigenvector PCA, Naidu & Raol (2008)
`bayesian_immerkaer` — inverse-variance precision weighting, Eq. (5)
`proposed` — proposed saliency scheme, per-pixel local variance (15×15), Eq. (3)-(4)
`gff` — guided-filtering fusion, Li, Kang & Hu (2013) (added baseline)
`dwt` — discrete wavelet fusion, max-abs detail (added baseline)
Metrics (`src/metrics.py`)
In paper: `EN` (entropy), `MI` (summed mutual information), `SSIM` (averaged)
Added: `Qabf` (Xydeas–Petrovic gradient quality), `VIF` (pixel-domain visual
information fidelity), `SD` (contrast), `SF` (spatial frequency)
Statistics (`src/evaluate.py`)
Per-image CSV, mean ± std summary tables, paired t-tests and Cohen's d_z
(proposed vs each baseline), and the mean per-pixel IR weight that documents
the saliency-vs-precision inversion.
Figure 3 (`src/make_figure3_closedloop.py`)
Regenerates the closed-loop convergence figure exactly as specified in §6.3.
---
Installation
```bash
python -m venv venv && source venv/bin/activate   # optional
pip install -r requirements.txt
```
Data
Download the public datasets (not redistributed here):
RoadScene — https://github.com/hanna-xu/RoadScene
MSRS — https://github.com/Linfeng-Tang/MSRS
Default expected layout (override with `--vis-dir` / `--ir-dir`):
```
data/RoadScene/crop_LR_visible/*.jpg   data/RoadScene/cropinfrared/*.jpg   # aligned same-size pairs
data/MSRS/test/vi/*.png      data/MSRS/test/ir/*.png   # suffix D=day, N=night
```
Visible/infrared pairs are matched by identical file stem; visible images are
converted to luminance and resized to the infrared grid automatically.
Running
```bash
# RoadScene (all 221 pairs)
python src/evaluate.py --dataset roadscene \
    --vis-dir data/RoadScene/crop_LR_visible --ir-dir data/RoadScene/cropinfrared

# MSRS daytime (reproduces the in-paper subset)
python src/evaluate.py --dataset msrs --time-of-day day \
    --vis-dir data/MSRS/test/vi --ir-dir data/MSRS/test/ir

# MSRS nighttime (the extension recommended by review)
python src/evaluate.py --dataset msrs --time-of-day night \
    --vis-dir data/MSRS/test/vi --ir-dir data/MSRS/test/ir

# Limit to a subset of methods
python src/evaluate.py --dataset roadscene --methods proposed bayesian_immerkaer gff \
    --vis-dir data/RoadScene/crop_LR_visible --ir-dir data/RoadScene/cropinfrared
```
Each run writes `results_<dataset>_<tod>.csv` (one row per image per method) and
prints the summary and paired-statistics tables to stdout.
Notes on reproducing the manuscript numbers
The four in-paper methods (`fixed_weight`, `pca`, `bayesian_immerkaer`,
`proposed`) reproduce the Table 4–6 quantities. Small differences (<~1%) from
the published values can arise from interpolation/library versions; if you
used a specific resize or histogram-bin convention in your original runs, set
it here to match.
`Qabf`, `VIF`, `SD`, `SF`, the `gff`/`dwt` baselines, and the MSRS-night run
are new: their numbers are produced by this code, not taken from the
current manuscript. Add them to the tables/text only after you have run them.
`VIF` here is the pixel-domain estimator; if a reviewer requests the
fusion-specific VIFF (Han et al., 2013), swap in a reference VIFF and re-run.
Precomputed results
`results/` contains the per-image CSVs that back Tables 4–7 of the manuscript
(`results_roadscene.csv`, `results_msrs_day.csv`, `results_msrs_night.csv`) and
`analyze.py`, which regenerates the summary tables, the weight-inversion table,
and the paired effect sizes from those CSVs:
```bash
python results/analyze.py
```
Archiving (to obtain the real Data Availability links)
Create a public GitHub repository and push this package.
Link the repo to Zenodo and cut a release to mint a DOI
(https://docs.github.com/en/repositories/archiving-a-github-repository/referencing-and-citing-content).
Put the GitHub URL and Zenodo DOI into the manuscript's Data Availability
statement, replacing the placeholder text.
License
MIT (see `LICENSE`).
