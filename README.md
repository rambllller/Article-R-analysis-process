# Metagenomic workflow for MetaCyc pathway and buglist analyses

This repository provides a publication-ready workflow starting from `clean_fastq` files.

It contains two parallel analysis branches:

1. **MetaCyc pathway branch**  
   `clean_fastq -> HUMAnN3 -> *_pathabundance.tsv -> merged MetaCyc matrix -> LEfSe -> Maaslin2`
2. **Buglist taxonomy branch**  
   `clean_fastq -> MetaPhlAn -> *_metaphlan_bugs_list.tsv -> merged buglist matrix -> LEfSe -> Maaslin2`

## Important note about Maaslin
The original scripts supplied for this repository were named as "MaAsLin3" scripts, but the actual code used `library(Maaslin2)` and `Maaslin2(...)`. For reproducibility, this repository keeps the implementation as **Maaslin2** and names the scripts accordingly.

If your manuscript currently says **MaAsLin3**, either:
- revise the manuscript text to **Maaslin2**, or
- replace these scripts with a true MaAsLin3 implementation and regenerate the results.

## Repository structure

```text
metagenome_github_repo/
├── README.md
├── .gitignore
├── metadata/
│   └── groups.tsv.example
└── scripts/
    ├── 01_run_humann_metacyc.sh
    ├── 02_merge_metacyc.py
    ├── 03_lefse_metacyc.R
    ├── 04_maaslin2_metacyc.R
    ├── 05_run_metaphlan_buglist.sh
    ├── 06_merge_buglist.py
    ├── 07_lefse_buglist.R
    ├── 08_maaslin2_buglist.R
    └── 09_run_all.sh
```

## Minimal files required for GitHub upload

### Core reproducibility files: **11 files**
1. `README.md`
2. `.gitignore`
3. `metadata/groups.tsv.example`
4. `scripts/01_run_humann_metacyc.sh`
5. `scripts/02_merge_metacyc.py`
6. `scripts/03_lefse_metacyc.R`
7. `scripts/04_maaslin2_metacyc.R`
8. `scripts/05_run_metaphlan_buglist.sh`
9. `scripts/06_merge_buglist.py`
10. `scripts/07_lefse_buglist.R`
11. `scripts/08_maaslin2_buglist.R`

### Optional convenience file
12. `scripts/09_run_all.sh`

So the **minimum recommended upload is 11 files**, and the **complete convenient version is 12 files**.

## Input requirements

### 1. Clean FASTQ files
Place all cleaned reads in one directory. File names must match one of these patterns:
- `SRRxxxx.clean.fastq`
- `SRRxxxx.clean.fq`
- `SRRxxxx.clean.fastq.gz`
- `SRRxxxx.clean.fq.gz`

### 2. Group file
Prepare a tab-delimited metadata file like this:

```tsv
SampleID	Group
SRR000001	NO
SRR000002	YES
```

- `SampleID` must match the sample names parsed from FASTQ names.
- For the current LEfSe scripts, **two-group comparison is required**.
- The reference group for the examples below is `NO`.

## Software requirements

### Server-side
- Bash
- Python 3
- HUMAnN3
- MetaPhlAn
- pandas

### R-side
- data.table
- dplyr
- tibble
- stringr
- ggplot2
- SummarizedExperiment
- S4Vectors
- lefser
- Maaslin2

The R scripts will try to install missing packages automatically when possible.

## Branch A: MetaCyc pathway workflow

### Step 1. Run HUMAnN3

```bash
bash scripts/01_run_humann_metacyc.sh \
  -i /path/to/clean_fastq \
  -o /path/to/output/humann_metacyc \
  -t 8 \
  -n /path/to/chocophlan \
  -p /path/to/uniref \
  -m /path/to/metaphlan_databases
```

Outputs per sample:
- `*_genefamilies.tsv`
- `*_pathabundance.tsv`
- `*_pathcoverage.tsv`

### Step 2. Merge MetaCyc pathway tables

```bash
python3 scripts/02_merge_metacyc.py \
  --input-dir /path/to/output/humann_metacyc \
  --output /path/to/output/humann_metacyc/metacyc_pathway_matrix_with_annotation.tsv
```

### Step 3. LEfSe on MetaCyc matrix

```bash
Rscript scripts/03_lefse_metacyc.R \
  --input-matrix /path/to/output/humann_metacyc/metacyc_pathway_matrix_with_annotation.tsv \
  --group-file /path/to/groups.tsv \
  --outdir /path/to/results/lefse_metacyc \
  --sample-column SampleID \
  --group-column Group \
  --ref-level NO \
  --case-level YES
```

### Step 4. Maaslin2 on MetaCyc matrix

```bash
Rscript scripts/04_maaslin2_metacyc.R \
  --input-matrix /path/to/output/humann_metacyc/metacyc_pathway_matrix_with_annotation.tsv \
  --group-file /path/to/groups.tsv \
  --outdir /path/to/results/maaslin2_metacyc \
  --sample-column SampleID \
  --group-column Group \
  --reference-level NO \
  --q-cutoff 0.2
```

## Branch B: Buglist taxonomy workflow

### Step 1. Run MetaPhlAn directly from clean FASTQ

```bash
bash scripts/05_run_metaphlan_buglist.sh \
  -i /path/to/clean_fastq \
  -o /path/to/output/buglist \
  -t 8 \
  -m /path/to/metaphlan_databases
```

Output per sample:
- `*_metaphlan_bugs_list.tsv`

### Step 2. Merge buglist tables

```bash
python3 scripts/06_merge_buglist.py \
  --input-dir /path/to/output/buglist \
  --output /path/to/output/buglist/bugList_abundance.tsv
```

### Step 3. LEfSe on buglist matrix

```bash
Rscript scripts/07_lefse_buglist.R \
  --input-abundance /path/to/output/buglist/bugList_abundance.tsv \
  --group-file /path/to/groups.tsv \
  --outdir /path/to/results/lefse_buglist \
  --sample-column SampleID \
  --group-column Group \
  --ref-level NO \
  --case-level YES
```

### Step 4. Maaslin2 on buglist matrix

```bash
Rscript scripts/08_maaslin2_buglist.R \
  --input-abundance /path/to/output/buglist/bugList_abundance.tsv \
  --group-file /path/to/groups.tsv \
  --outdir /path/to/results/maaslin2_buglist \
  --sample-column SampleID \
  --group-column Group \
  --reference-level NO \
  --q-cutoff 0.2
```

## One-command convenience runner

If you want a wrapper for both branches, edit variables in:

```bash
bash scripts/09_run_all.sh
```

## Recommended output directories

```text
results/
├── humann_metacyc/
├── buglist/
├── lefse_metacyc/
├── maaslin2_metacyc/
├── lefse_buglist/
└── maaslin2_buglist/
```

## What you should NOT upload to GitHub

Do not upload:
- raw FASTQ files
- huge HUMAnN3 intermediate files
- full result directories containing large binary files
- absolute local paths from your workstation or server

Upload only the workflow code, example metadata, and a short README.

## What is still missing and should be checked by you

Before final submission, please check these items:
1. Whether your manuscript should say **Maaslin2** or **MaAsLin3**.
2. Whether your actual group labels are really `NO` and `YES`.
3. Whether you want to keep only unstratified MetaCyc pathways.
4. Whether the server-side software versions should be documented in the manuscript or supplementary methods.
5. Whether you want to add a `sessionInfo()` text file from R and `humann --version` / `metaphlan --version` outputs as supplementary reproducibility records.

