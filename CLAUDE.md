# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SigProfilerTopography (SPT) is a Python package for analyzing the topography of mutational signatures in cancer genomes. It is part of the SigProfiler suite and analyzes mutations with respect to nucleosome occupancy, epigenomics occupancy, replication timing, strand asymmetry, and strand-coordinated mutagenesis. Supports SBS, DBS, and ID mutation types on GRCh37, GRCh38, mm9, and mm10 genomes.

## Build and Install

```bash
pip install .                          # Install from source
pip install SigProfilerTopography      # Install from PyPI
```

Version is defined in `setup.py` (`VERSION` variable) and auto-generated into `SigProfilerTopography/version.py` during installation.

## Testing

```bash
python3 tests/spt_test.py
```

Tests require the GRCh37 reference genome to be installed first:
```bash
python -c "from SigProfilerMatrixGenerator import install as genInstall; genInstall.install('GRCh37')"
```

There is no pytest configuration; the test file imports and runs the package directly. CI uses Travis CI (`.travis.yml`).

## Architecture

### Entry Point

`SigProfilerTopography/Topography.py` — Contains `runAnalyses()` (line ~1240), the main orchestration function with ~100 parameters controlling which analyses to run. Also provides `install_*()` functions for downloading reference data (nucleosome, ATAC-seq, Repli-seq, example data).

### Pipeline Steps (controlled by `step1`–`step5` parameters in `runAnalyses`)

1. **step1_matgen_real_data**: Run SigProfilerMatrixGenerator on real mutations
2. **step2_gen_sim_data**: Run SigProfilerSimulator to generate null distributions
3. **step3_matgen_sim_data**: Run SigProfilerMatrixGenerator on simulated mutations
4. **step4_merge_prob_data**: Merge mutations with signature probability assignments
5. **step5_gen_tables**: Generate summary tables for signatures

### Analysis Modules (under `SigProfilerTopography/source/`)

| Module | Purpose |
|---|---|
| `commons/TopographyCommons.py` | Central constants, helpers, data I/O (~5000 lines) |
| `occupancy/` | Nucleosome and epigenomics occupancy analysis |
| `replicationtime/` | Normalized mutation density by replication timing deciles |
| `replicationstrandbias/` | Replication strand asymmetry (lagging vs leading) |
| `transcriptionstrandbias/` | Transcription strand asymmetry, genic vs intergenic |
| `processivity/` | Strand-coordinated mutagenesis (consecutive mutation distances) |
| `annotatedregion/` | Genomic region annotation analysis |
| `annotation/` | Mutation annotation, VEP integration |
| `plotting/` | Visualization (occupancy figures, replication timing, strand bias, processivity) |
| `fonts/` | Font resources for plots |

### Data Flow

Input VCFs → SigProfilerMatrixGenerator (matrices) → SigProfilerAssignment/user probabilities (signature assignment) → SigProfilerSimulator (null distributions) → Topography analyses → Output tables + figures

### Key Dependencies

- **SigProfiler suite**: SigProfilerMatrixGenerator, SigProfilerSimulator, SigProfilerAssignment
- **Scientific**: numpy, pandas, scipy, statsmodels, matplotlib
- **Genomics**: pyranges, intervaltree, pyBigWig

### Parallelization

Uses `multiprocessing.Pool` with `apply_async` for chromosome-based parallel processing. Controlled by the `parallel_mode` parameter in `runAnalyses()`.

### Library Data

`SigProfilerTopography/source/lib/` contains bundled reference data files (chromosome sizes, replication timing BED/WIG files, epigenomics signal files). Additional data is downloaded via `install_*()` functions.
