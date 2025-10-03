# Exp. 91 FTIR Analysis: Experimental Adhesives (R Scripts)

## Overview
This repository contains two R scripts used in the Exp. 91 project to assess FTIR spectral **reproducibility** and **cross-instrument comparability**. Both scripts read FTIR spectra provided as `.csv`, handle common header variations automatically, interpolate to align spectra, and write results back to `.csv` for downstream stats and figures.

- **Script 1 — Cross-Instrument Comparison (KBr/LN-MCT-B vs ZnSe/TE-MCT-A):** Pairs like-for-like spectra, interpolates to a common grid, and computes agreement metrics (Pearson *r*, CV, cosine similarity, Wilcoxon signed-rank, Bland–Altman).
- **Script 2 — Orientation test:** Within a specimen, compares **orientation-to-orientation** spectra **separately** for KKT and Raw datasets (no cross-mixing), with interpolation to a per-group grid and the same agreement metrics.

These tools underpin analyses for Lien et al. (in prep.), *FTIR Analysis of Experimental Adhesives: Investigating Spectral Reproducibility, Chemometric Approaches, and Archaeological Applications*.

---

## Data & Naming

### File format
All spectra are `.csv`. The scripts auto-detect typical headers:
- **Wavenumber:** `Wavenumber`, `Wavenumbers`
- **Signal (absorbance):** `Absorbance`, `SingleBeam`, `single_beam`, or `Interferogram`  
Values are parsed to numeric; extra columns are ignored.

### Folder layout (example; adjust as needed)
    project-root/
    ├─ data/
    │  ├─ agilent_kkt/  # .csv spectra (Agilent, processed/KKT)
    │  ├─ agilent_raw/  # .csv spectra (Agilent, raw)
    │  ├─ bruker_kkt/   # .csv spectra (Bruker, processed/KKT)
    │  ├─ bruker_raw/   # .csv spectra (Bruker, raw)
    │  └─ repeated_scans/
    │     ├─ KKT/       # .csv spectra for repeated scans (KKT)
    │     └─ Raw/       # .csv spectra for repeated scans (Raw)
    └─ metadata/
       └─ exp_91_metadata_instrument_comparison.csv

### Naming conventions (used by the scripts for parsing)
- **Cross-instrument (generic):** Filenames become `SampleID`; matching across instruments is done via a cleaned stem (`MatchID`) derived from the first tokens of the filename.
- **Orientation test:**  
  `exp 91 <specimen> point 1 <sample-text> orientation <N> <…> [KKT|Raw].csv`  
  Examples: `exp 91 11 point 1 vegetal fiber orientation 1 2cm ll.csv`,  
  `exp 91 11 point 1 vegetal fiber orientation 2 2cm ll KKT.csv`.

---

## Scripts

### Script 1 — Cross-Instrument Comparison (KBr/LN-MCT-B vs ZnSe/TE-MCT-A, or Agilent vs Bruker)
**What it does**  
Reads `.csv` spectra from Agilent (KKT/Raw) and Bruker (KKT/Raw); infers wavenumber & absorbance columns; cleans filenames into `SampleID` and `MatchID`; interpolates each spectrum to a common wavenumber grid; pairs Agilent ↔ Bruker per `MatchID`; then computes:
- Pearson correlation (*r*)
- Coefficient of variation (CV) per instrument
- Cosine similarity
- Wilcoxon signed-rank (*paired*)
- Bland–Altman bias and limits of agreement

**Inputs (edit in the script)**  
- Paths for: `agilent_kkt/`, `agilent_raw/`, `bruker_kkt/`, `bruker_raw/`  
- Optional metadata CSV: `metadata/exp_91_metadata_instrument_comparison.csv` (joins on cleaned `sample_id`)

**Outputs (CSV, written alongside your metadata file)**  
- `summary_metrics.csv`, `cor_cv_cosine.csv`, `wilcoxon_results.csv`, `bland_altman.csv`  

---

### Script 2 — Orientation test: Orientation-to-Orientation
**What it does**  
Within each specimen, compares **orientation** spectra for **point 1** only. It runs **twice**, once for KKT and once for Raw (no cross-mixing). For each `(specimen × data_type)` group, spectra are averaged at duplicate wavenumbers, interpolated to a per-group **union** grid (gentle end extrapolation), and compared pairwise across orientations. Metrics reported:
- Pearson correlation (*r*)
- Cosine similarity
- Cohen’s *d* (paired)
- Wilcoxon signed-rank (*paired*)
- SD, mean difference, and CV of (x − y)

**Inputs (edit in the script)**  
- `dir_kkt` — folder containing KKT `.csv` files  
- `dir_raw` — folder containing Raw `.csv` files  
- *(Optional)* `specimen_filter <- c("11","26",...)` to restrict which specimens to include

**Outputs (CSV)**  
- `spectral_pairwise_metrics_orientations_point1_KKT.csv` (written to `dir_kkt`)  
- `spectral_pairwise_metrics_orientations_point1_Raw.csv` (written to `dir_raw`)

---

## Quick Start
1. Clone / download this repository.  
2. Place your data under the folders described above (or point the scripts to your own paths).  
3. Install packages (see below).  
4. Open R / RStudio and run:
   - Script A for cross-instrument comparisons  
   - Script B for orientation comparisons (KKT and Raw exported separately)

Each script stops with a clear message if no eligible files are found or if parsing fails, and prints diagnostics (e.g., number of orientations per specimen).

---

## Dependencies
Install these packages before running the scripts:

    install.packages(c(
      "dplyr", "tidyr", "stringr", "readr", "janitor",
      "ggplot2", "effsize", "purrr"
    ))

If your workflow includes additional visualization or statistical steps (e.g., PCA/HCA), you may also want: **factoextra**, **cluster**, and related plotting libraries.

---

## Tips & Troubleshooting
- **Headers differ?** The scripts already check `Wavenumber/Wavenumbers` and `Absorbance/SingleBeam/single_beam/Interferogram`. If you see “Skipping (no wavenumber/signal) …”, copy the first few column names and add another pattern to the detector.  
- **No results for Raw/KKT?** Ensure your `dir_raw` and `dir_kkt` paths point to real folders and your filenames include `exp 91 NN`, `point 1`, and `orientation N`.  
- **Metadata join (Script A):** The metadata CSV should contain a `sample_id` column that matches (after cleaning) the `SampleID` produced from filenames. If a metadata file isn’t present, the script continues without merging.  
- **Interpolation:** Both scripts interpolate to a common grid so vectors align when wavenumber spacing differs across instruments or runs.

---

## Citation
If you use these scripts, please cite:  
**Lien et al. (in preparation)**, *FTIR Analysis of Experimental Adhesives: Investigating Spectral Reproducibility, Chemometric Approaches, and Archaeological Applications.*

Copyright 2025 ULiege
[Author Lauren Lien <lauren.lien@uliege.be.>]

---

