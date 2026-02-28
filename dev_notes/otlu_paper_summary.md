# Paper Summary: Otlu & Alexandrov 2024

**"Evaluating topography of mutational signatures with SigProfilerTopography"**
bioRxiv, Jan 2024. DOI: 10.1101/2024.01.08.574683

## Overview

SigProfilerTopography (SPT) is a Python package for comprehensive profiling of the topography of mutational signatures of all small mutational events (SBS, DBS, ID). It evaluates how chromatin organization, histone modifications, transcription factor binding, DNA replication, and DNA transcription affect the activities of different mutational processes. Demonstrated on 552 whole-genome sequenced esophageal squamous cell carcinomas (ESCCs).

## Core Algorithmic Approach

All analyses follow a single paradigm: **compare real mutations against simulated null distributions**, then test for statistically significant differences with Benjamini-Hochberg FDR correction (q-value <= 0.05).

### Step 1: Null Model Generation

SigProfilerSimulator generates *n* simulated datasets (default *n*=100) per sample. The simulation preserves:
- Total mutation count per chromosome
- Trinucleotide context of each SBS (the mutated base + immediate 5' and 3' base-pairs)
- Overall mutational channel proportions

This creates a background model that accounts for genome structure while randomizing mutation positions.

### Step 2: Mutation Classification

Both real and simulated mutations are categorized by mutation type using SigProfilerMatrixGenerator.

### Step 3: Signature Probability Assignment

Each mutation (real and simulated) is probabilistically attributed to a mutational signature using:
- SigProfilerAssignment with COSMIC reference signatures (default), OR
- User-provided signature pattern + activity matrices

### Step 4: False-Positive Rate Control

Only mutations with >= 90% average probability of being caused by a specific mutational signature are included in downstream analyses.

### Step 5: Topographical Analyses (real vs. simulated comparisons)

## Five Analysis Types

### 1. Feature Occupancy Analysis

For each mutation, SPT extracts the signal from a topographical feature (e.g., nucleosome MNase-seq, CTCF ChIP-seq, histone marks) within a +/-1 kb flanking window centered on the mutation site. The signal is aggregated position-by-position across all mutations and averaged, producing an average-signal-vs-distance profile. The same procedure is performed for simulated mutations. Comparing the two profiles reveals enrichments or depletions.

**Example findings:**
- SBS17b shows ~190 bp nucleosome periodicity (high damage + reduced repair at nucleosome positions)
- SBS17b enriched at CTCF binding sites, depleted at histone marks (especially H3K4me1, H3K27ac enhancer marks)
- ID2 depleted at nucleosome-occupied regions, enriched at CTCF binding sites

**Summary heatmap:** A fold-change heatmap (real/simulated) summarizes enrichments (red) and depletions (blue) across all signatures and topographical features, with statistically significant results marked (q-value <= 0.05).

### 2. Replication Timing Analysis

1. Repli-seq weighted-average data is wavelet-smoothed
2. Local maxima in smoothed signal = replication initiation zones (early replicating)
3. Local minima = replication termination zones (late replicating)
4. Signal is sorted in descending order (early -> late) and split into 10 deciles
5. Normalized mutation density is computed per decile for each signature
6. Compared to the same metric computed from simulated mutations

**Example finding:** APOBEC signature SBS2 shows increasing normalized mutation density from early to late replicating regions.

### 3. Replication Strand Asymmetry

Using peaks (initiation zones) and valleys (termination zones) from wavelet-smoothed Repli-seq:
1. The slope of the replication signal at each mutation position determines strand assignment:
   - Positive slope region: + strand = leading, - strand = lagging
   - Negative slope region: + strand = lagging, - strand = leading
2. Mutations are oriented by the pyrimidine base of the Watson-Crick base-pair
3. Counts on leading vs lagging strands are compared to simulated data
4. Reported separately for each of the six substitution subtypes (C>A, C>G, C>T, T>A, T>C, T>G)

**Example finding:** SBS2 (APOBEC) shows enrichment on the lagging strand, consistent with APOBEC deaminases targeting single-stranded DNA during replication.

### 4. Transcription Strand Asymmetry

1. Mutations are classified as **genic** (within protein-coding genes) vs **intergenic** (outside)
2. Genic mutations are further subclassified as **transcribed-strand** or **un-transcribed-strand** by orienting the pyrimidine base of the reference Watson-Crick base-pair relative to gene direction
3. Counts on each strand and in each region are compared to simulated data per substitution subtype

**Example finding:** SBS16 (alcohol-associated) shows higher T>C mutations on the transcribed strand and enrichment in genic regions, attributed to transcription-coupled damage in actively transcribed genes.

### 5. Strand-coordinated Mutagenesis (Processivity)

1. Consecutive SBS mutations attributed to the same signature are identified
2. Must be on the same strand (oriented by reference Watson-Crick base)
3. Must be within <= 10 kb of each other
4. Groups of varying lengths (2, 3, ..., up to 11) are pooled across all samples per signature
5. The same grouping is performed on simulated mutations
6. Statistical significance of observed group counts is tested at each length

**Example findings:**
- APOBEC signatures SBS2 and SBS13: groups of up to 11 consecutive mutations (kataegis)
- Flat signatures SBS5 and SBS40: strand-coordinated mutagenesis of varying group length
- Mismatch repair deficiency SBS15: groups up to 5 consecutively mutated bases

## Input Formats

- **Mutations:** VCF, MAF, ICGC, simple text
- **Topographical features:** wig, bigWig, bed, bigBed

## Statistical Framework

- All comparisons are real vs. simulated
- Benjamini-Hochberg correction for multiple hypothesis testing
- Uses the statsmodels Python package
- q-value <= 0.05 threshold for significance

## Known Limitations

1. Only supports small mutational events (SBS, DBS, ID) — not copy-number changes or structural rearrangements
2. Requires whole-genome sequencing data (not whole-exome or targeted panels) since the algorithm profiles non-coding regions
3. Requires sufficient numbers of somatic mutations for statistical power
