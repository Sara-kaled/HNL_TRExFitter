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
