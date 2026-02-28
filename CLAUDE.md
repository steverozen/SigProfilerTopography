# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SigProfilerTopography (SPT) is a Python package for analyzing the topography of mutational signatures in cancer genomes. It is part of the SigProfiler suite and analyzes mutations with respect to nucleosome occupancy, epigenomics occupancy, replication timing, strand asymmetry, and strand-coordinated mutagenesis. Supports SBS, DBS, and ID mutation types on GRCh37, GRCh38, mm9, and mm10 genomes.

## Git Workflow

Always push to `origin` (`steverozen/SigProfilerTopography`), never to `upstream` (`SigProfilerSuite/SigProfilerTopography`) — no write access to upstream.

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

## Algorithm Details (from Otlu & Alexandrov 2024, bioRxiv 10.1101/2024.01.08.574683)

The core algorithmic approach across all analyses is **real-vs-simulated comparison**: every topographical feature is evaluated for real somatic mutations and for simulated (null) mutations, and differences are tested statistically with Benjamini-Hochberg FDR correction (q-value ≤ 0.05).

### Simulation-based null model
SigProfilerSimulator generates *n* simulated datasets (default *n*=100) per sample. The simulation preserves: total mutation count per chromosome, trinucleotide context of each SBS, and overall mutational channel proportions. This creates a background model that accounts for genome structure while randomizing mutation positions.

### Signature probability assignment
Each mutation (real and simulated) is probabilistically attributed to a mutational signature using SigProfilerAssignment (COSMIC reference signatures) or user-provided signature/activity matrices. Only mutations with ≥90% average probability of belonging to a specific signature are included in downstream analyses (controls false-positive rate).

### Feature Occupancy Analysis
For each mutation, the tool extracts signal from the topographical feature (e.g., nucleosome MNase-seq, CTCF ChIP-seq, histone marks) within a ±1 kb flanking window. Signals are aggregated position-by-position across all mutations and averaged, producing an average-signal-vs-distance profile centered on the mutation site. The same is done for simulated mutations. Comparing profiles reveals enrichments/depletions (e.g., ~190 bp nucleosome periodicity for SBS17b, CTCF enrichment at mutation sites).

### Replication Timing Analysis
Repli-seq weighted-average data is wavelet-smoothed. Local maxima = replication initiation zones (early), local minima = termination zones (late). The smoothed signal is sorted descending (early→late) and split into 10 deciles. Normalized mutation density is computed per decile for each signature, and compared to simulated data.

### Replication Strand Asymmetry
Using peaks/valleys from the wavelet-smoothed Repli-seq signal, mutations are annotated as leading or lagging strand based on: (1) the slope of the replication signal at the mutation position (positive slope = leading on + strand, lagging on − strand; negative slope = vice versa), and (2) orienting by the pyrimidine base of the Watson-Crick base-pair. Counts of mutations on leading vs lagging strands are compared to simulated data for each of the six substitution subtypes (C>A, C>G, C>T, T>A, T>C, T>G).

### Transcription Strand Asymmetry
Mutations in protein-coding genes are classified as genic vs intergenic. Genic mutations are further subclassified as transcribed-strand or un-transcribed-strand by orienting the pyrimidine base of the reference Watson-Crick base-pair relative to gene direction. Counts on each strand and in each region are compared to simulated data per substitution subtype.

### Strand-coordinated Mutagenesis (Processivity)
Consecutive SBS mutations attributed to the same signature, on the same strand (oriented by reference Watson-Crick base), and within ≤10 kb of each other are grouped. Groups of length 2, 3, ..., up to 11 are pooled across all samples per signature. The same grouping is performed on simulated mutations to assess statistical significance of the observed group counts at each length.
