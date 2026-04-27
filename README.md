# QIIME2 Analysis of Saffron Rhizosphere Microbiome

## Project Overview

This repository contains a complete QIIME2-based workflow for 16S rRNA amplicon metagenomics analysis of saffron rhizosphere samples from different geographical locations.

The dataset used in this project is publicly available from NCBI BioProject:

**BioProject ID:** PRJNA687631
**Title:** Saffron Rhizosphere bacteriome from different geographical locations

The main objective of this study is to investigate microbial diversity, taxonomic composition, and comparative bacteriome structure across environmental samples using QIIME2.

This project was developed as part of MSc Computational Biology research focused on metagenomics analysis of environmental samples.

---

## Dataset Information

* **Database:** NCBI BioProject
* **BioProject ID:** PRJNA687631
* **Platform:** Illumina
* **Data Type:** 16S rRNA Amplicon Sequencing
* **Number of Samples:** 10
* **Study Focus:** Rhizosphere microbiome diversity analysis

Dataset Link: https://www.ncbi.nlm.nih.gov/bioproject/PRJNA687631

---

## Workflow Overview

Raw FASTQ Files
↓
Import into QIIME2 (.qza)
↓
Sequence Quality Control using DADA2
↓
Feature Table Construction
↓
Phylogenetic Tree Generation
↓
Alpha Diversity Analysis
↓
Beta Diversity Analysis
↓
Alpha Rarefaction Plotting
↓
Taxonomic Classification
↓
Visualization and Interpretation

---

## Analysis Steps

### 1. Import FASTQ Data - Import Data via Manifest File

First, create a manifest file (`manifest.tsv`) using the following format:

```tsv
sample-id	forward-absolute-filepath	reverse-absolute-filepath
sample-1	/path/to/project/sample1_forward.fastq.gz	/path/to/project/sample1_reverse.fastq.gz
sample-2	/path/to/project/sample2_forward.fastq.gz	/path/to/project/sample2_reverse.fastq.gz
sample-3	/path/to/project/sample3_forward.fastq.gz	/path/to/project/sample3_reverse.fastq.gz
sample-4	/path/to/project/sample4_forward.fastq.gz	/path/to/project/sample4_reverse.fastq.gz
sample-5	/path/to/project/sample5_forward.fastq.gz	/path/to/project/sample5_reverse.fastq.gz
sample-6	/path/to/project/sample6_forward.fastq.gz	/path/to/project/sample6_reverse.fastq.gz
sample-7	/path/to/project/sample7_forward.fastq.gz	/path/to/project/sample7_reverse.fastq.gz
sample-8	/path/to/project/sample8_forward.fastq.gz	/path/to/project/sample8_reverse.fastq.gz
sample-9	/path/to/project/sample9_forward.fastq.gz	/path/to/project/sample9_reverse.fastq.gz
sample-10	/path/to/project/sample10_forward.fastq.gz	/path/to/project/sample10_reverse.fastq.gz
```

Replace `/path/to/project/` with the actual absolute path to your FASTQ files on your local system.

After creating the manifest file, import the paired-end sequence data into QIIME2 artifact format (`.qza`) using:

```bash
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path  /path/to/manifest.tsv \
  --output-path paired-end-demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
```

This will generate the imported QIIME2 artifact file:

```text
paired-end-demux.qza
```

### 2. Quality Control using DADA2

DADA2 was used for:

* Quality filtering
* Denoising
* Chimera removal
* Feature table generation
* Representative sequence generation

### 3. Phylogenetic Tree Construction

A phylogenetic tree was generated for phylogenetic diversity analysis using:

* Multiple sequence alignment
* Masking
* Tree construction
* Rooted tree generation

### 4. Alpha Diversity Analysis

Alpha diversity metrics were calculated to evaluate within-sample diversity.

Examples:

* Shannon Diversity Index
* Observed Features
* Faith’s Phylogenetic Diversity
* Pielou’s Evenness

### 5. Beta Diversity Analysis

Beta diversity analysis was performed to compare microbial community composition between samples.

Examples:

* Bray-Curtis Distance
* Jaccard Distance
* UniFrac Analysis
* PCoA Visualization

### 6. Alpha Rarefaction

Rarefaction analysis was performed to evaluate sequencing depth sufficiency.

### 7. Taxonomic Classification

Taxonomic assignment was performed to identify microbial composition across samples.

### 8. Visualization

Final results were visualized using:

* Taxonomy barplots
* Rarefaction plots
* Alpha diversity plots
* PCoA plots
* Heatmaps
* Comparative abundance graphs

---

## Repository Structure

```text
QIIME2-Analysis/
│
├── README.md
├── requirements.txt
├── environment.yml
├── metadata/
├── scripts/
├── results/
├── figures/
├── docs/
└── LICENSE
```

---

## Tools Used

* QIIME2
* DADA2
* SILVA Database
* Python
* Bash
* Linux Command Line
* R (for visualization if applicable)

---

## Future Improvements

* Comparative analysis using Kraken2
* Integration with machine learning classifiers
* DNABERT-based microbiome classification
* Deep learning-based taxonomic prediction

---

## Author

**Mohamed Jasee**
MSc Computational Biology
Bioinformatics | Metagenomics | NGS Analysis | Microbiome Research

---

## License

This project is for academic and research purposes.
