# HNL_Statistical

This repository contains the statistical analysis framework for the Heavy Neutral Lepton (HNL) search, utilizing the **TRExFitter** package.

## Building TRExFitter from Source

If you need to build TRExFitter manually rather than using a pre-compiled version, follow the steps below.

### 1. Prerequisites
Assuming your machine uses **EL9 OS** and has access to **cvmfs** (e.g., `lxplus`), start by setting up the ATLAS environment:

```bash
setupATLAS
lsetup git
```

### 2. Obtaining the Package

Clone the repository with submodules and install Git LFS for large file handling:
 Clone the repository (recursive ensures submodules are included)
```bash
git clone --recursive ssh://git@gitlab.cern.ch:7999/TRExStats/TRExFitter.git
cd TRExFitter
```

## Install extension for large files used in CI tests
```bash
git lfs install
cd -
```

## If it is your first time cloning, or if submodules have changed later, run:
```bash
cd TRExFitter
git submodule init
git submodule update
```

## This step is essential to set the correct paths and aliases for your local version. You must be inside the TRExFitter directory.
```bash
source setup.sh
```


# TRExFitter Commands Guide for HNL_run2_nominal.config

## Introduction

TRExFitter is a framework for building and performing statistical tests with high-energy physics data using HistFactory models. This README explains the core commands using the example configuration file `config/HNL_run2_nominal.config`. These commands guide you through the workflow from input histogram production to advanced analyses like limits, significance, and ranking plots.

**Prerequisites:**
- TRExFitter installed and `trex-fitter` executable available in PATH.
- Configuration file `config/HNL_run2_nominal.config` (nominal) or `config/HNL_run2_allSYS.config`(all systematics) prepared (defines Jobs, Regions, Samples, Systematics, etc.).
- Copy HNL_run2_nominal.config into TRExFitter/config directory
- Input ntuples accessible (e.g., on AFS).

All commands follow the pattern:
```
trex-fitter [action] config/HNL_run2_nomial.config
```
Running a command creates/updates a folder named after the Job block in the config (e.g., `HNL_run2_nominal/`).

**Tip:** Increase `DebugLevel: 1` or `2` in the config for more verbose output. Re-run previous steps after config changes if needed.

## Command Usage

### 1. `trex-fitter n config/HNL_run2_nominal.config` - Input Histogram Production
**Purpose:** Read ntuples and produce histograms for further use.

**What happens:**
- Processes ntuples into histograms based on config specifications.
- Creates `Histograms/` folder with per-sample, per-region histograms.
- Protects empty bins by setting small finite yields (warning issued).

**Outputs:**
- `Histograms/` folder.
- Pruning info if applicable.

**Example time:** A few seconds.

**Tips:** Avoid empty bins by grouping samples or adjusting binning.

### 2. `trex-fitter w config/HNL_run2_nominal.config` - Create Workspace
**Purpose:** Produce RooStats workspace for the fit model.

**What happens:**
- Builds HistFactory model from histograms.
- Applies pruning (SystPruningShape/Norm) to neglect small-impact systematics.
- Skips validation regions.

**Outputs:**
- `RooStats/` folder with model files.
- Pruning reports (e.g., `SystPruningShape`, `Pruning.png` plot showing systematics impact per sample/region).
- Use `KeepPruning: TRUE` to skip pruned systematics in future runs.

**Tuning:** Adjust `SystPruningShape`, `SystPruningNorm`, `SystLarge`.

### 3. `trex-fitter d config/HNL_run2_nominal.config` - Pre-fit Plots
**Purpose:** Visualize regions before fitting.

**What happens:**
- Produces data/MC plots with systematics bands.

**Outputs:**
- `Plots/` folder: Per-region plots, summary plots.
- `PieChart.png`, `SignalRegions.png`: Background composition, signal fraction.
- `Tables/` folder: Yields, LaTeX tables.

### 4. `trex-fitter f config/HNL_run2_nominal.config` - Run Fit
**Purpose:** Perform the fit (default: signal+background).

**What happens:**
- Fits model to data.
- Computes best-fit parameters, pulls, correlations.

**Outputs:**
- `CorrMatrix.png`: Nuisance correlations (> threshold).
- `Gammas.png`, `NormFactors.png`, `Pulls/All/NuisPar.png`.
- `Fits/` folder: Best-fit results, correlations.
- Pull significances in `Pulls/PullSig/` and `PullSig/RankedPullSig/` (σ = |post-fit| / sqrt(1 - (error)^2)).

**Background-only:** Set `FitType: backgroundonly` in Fit block.

### 5. `trex-fitter p config/HNL_run2_nominal.config` - Post-fit Plots
**Purpose:** Visualize model after fit.

**What happens:**
- Applies best-fit nuisance parameters.
- Propagates uncertainties with correlations.

**Outputs:**
- Updated `Plots/` and `Tables/` with post-fit versions.
- Enable `UseGammaPulls: TRUE` to include gamma effects.

**Note:** Negative signal strength hides signal in plots.

### 6. `trex-fitter l config/HNL_run2_nominal.config` - Limits
**Purpose:** Calculate observed/expected upper limits (CLs method).

**What happens:**
- Uses asymptotic approximation.

**Outputs:**
- `Limits/asymptotics/myLimit_CL95.root` (stats tree).

**Signal injection:** `SignalInjection: TRUE` in Limit block.

### 7. `trex-fitter s config/HNL_run2_nominal.config` - Significance
**Purpose:** Compute observed/expected significance.

**Outputs:**
- `Significance/asymptotics/mySignificance_p0.root` (p0 tree).

### 8. `trex-fitter r config/HNL_run2_nominal.config` - Ranking Plot
**Purpose:** Rank nuisance parameters by impact on signal strength (μ).

**What happens:**
- Runs 4 fits per nuisance: ± pre-fit/post-fit shifts.
- Computes impact as Δμ.
- Parallelizable for complex configs.

**Outputs:**
- `RankingSysts.png`, `Ranking.png` (top impacts).
- Additional fit results in `Fits/`.

**Time:** Can be long; limit with `RankingMaxNP`.

## Additional Features

### Systematics Control Plots
- Set `SystControlPlots: TRUE` in Job block.
- Re-run `n` or `b`: Creates `Systematics/` with per-systematic plots (shapes, symmetrization, smoothing).
- Check `SystErrorBars` for stat uncertainties.

### Blinding
- Use `BlindingThreshold` to hide data in signal-enriched bins.

### Binning
- `AutoBin` for MVA outputs (TransfoD/F/J algorithms).

### Smoothing & Pruning
- Verify in `Systematics/` plots.


### Asimov Dataset
- Set `FitBlind: TRUE` in Fit block.
- Re-run `f`: Fits pseudo-data from nominal predictions (best-fits at nominal).

## Troubleshooting & Best Practices
- **Empty bins:** Group samples, refine binning.
- **Pruning issues:** Check `Pruning.png`.
- **Smoothing:** Inspect `Systematics/` for unphysical shapes.
- **Correlations:** `CorrelationThreshold` in plots.
- **Parallel ranking:** See TRExFitter README.
- Re-run sequence: `n → w → d → f → p → l → s → r`.


