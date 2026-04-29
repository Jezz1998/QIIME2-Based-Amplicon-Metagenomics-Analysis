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
After generating the imported QIIME2 artifact file (`paired-end-demux.qza`), it is useful to generate a summary of the sequence data.

This helps to:

* Determine how many sequences were obtained per sample
* Examine the distribution of sequence quality scores at each position
* Decide appropriate trimming and truncation parameters for DADA2 denoising

Use the following command to generate the summary visualization:

```bash
qiime demux summarize \
  --i-data /path/to/paired-end-demux.qza \
  --o-visualization demux.qzv
```

This will generate the visualization file:

```text
demux.qzv
```
You can view this file using QIIME2 View:

https://view.qiime2.org

### 2. Sequence Quality Control and Feature Table Construction using DADA2

Multiple denoising parameter combinations were tested to optimize sequence retention and quality filtering.
The following parameters produced the maximum yield and were selected for downstream analysis.

DADA2 was used for:

Quality filtering
Denoising
Chimera removal
Feature table generation
Representative sequence generation

Run the following command:

qiime dada2 denoise-paired \
  --i-demultiplexed-seqs paired-end-demux.qza \
  --p-trim-left-f 40 \
  --p-trim-left-r 40 \
  --p-trunc-len-f 0 \
  --p-trunc-len-r 285 \
  --o-representative-sequences representative-sequences.qza \
  --o-table table.qza \
  --o-denoising-stats denoising-stats.qza

This will generate:

representative-sequences.qza
table.qza
denoising-stats.qza

### 3. Feature Table and Feature Data Summaries

After the quality filtering step is completed, it is important to explore the resulting data using summary visualizations.

These summaries help to:

* Determine how many sequences are associated with each sample
* Examine how many sequences are associated with each feature (ASV)
* View histograms of feature and sample distributions
* Access summary statistics of the feature table
* Map feature IDs to their representative sequences
* Easily BLAST representative sequences against the NCBI nt database
* Evaluate DADA2 denoising performance, including:

  * Percentage of sequences passing quality filtering
  * Denoised reads
  * Merged reads
  * Non-chimeric reads from total input sequences

## a. DADA2 Denoising Statistics Summary

```bash id="ngg2cw"
qiime metadata tabulate \
  --m-input-file denoising-stats.qza \
  --o-visualization denoising-stats.qzv
```

## b. Feature Table Summary

```bash id="xvweja"
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv
```

## c. Representative Sequences Summary

```bash id="nmwbm6"
qiime feature-table tabulate-seqs \
  --i-data representative-sequences.qza \
  --o-visualization representative-sequences.qzv
```

This will generate:

```text id="g6u6ho"
denoising-stats.qzv
table.qzv
representative-sequences.qzv
```


### 4. ## Generate a Phylogenetic Tree for Diversity Analysis

QIIME2 supports several phylogenetic diversity metrics, including:

* Faith’s Phylogenetic Diversity
* Weighted UniFrac
* Unweighted UniFrac

In addition to the feature count table (`FeatureTable[Frequency]`), these analyses require a rooted phylogenetic tree that describes the evolutionary relationships among the observed features (ASVs).

This information is stored as a `Phylogeny[Rooted]` QIIME2 artifact.

To generate the phylogenetic tree, the `align-to-tree-mafft-fasttree` pipeline from the `q2-phylogeny` plugin was used.

This pipeline performs the following steps:

1. Multiple sequence alignment using **MAFFT**
2. Masking highly variable positions to reduce noise
3. Tree construction using **FastTree**
4. Midpoint rooting to generate a rooted phylogenetic tree

Run the following command:

```bash id="m0ajzy"
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences representative-sequences.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

This will generate:

```text id="prz4y9"
aligned-rep-seqs.qza
masked-aligned-rep-seqs.qza
unrooted-tree.qza
rooted-tree.qza
```

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
