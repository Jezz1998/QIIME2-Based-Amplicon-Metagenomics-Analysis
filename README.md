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


### 4. Generate a Phylogenetic Tree for Diversity Analysis

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

### 5. Alpha and Beta Diversity Analysis

QIIME2 diversity analyses were performed using the `q2-diversity` plugin, which supports:

* Alpha diversity metric calculation
* Beta diversity metric calculation
* Statistical testing
* Interactive PCoA visualization using Emperor

The `core-metrics-phylogenetic` method was used to rarefy the feature table to a fixed sampling depth, calculate multiple diversity metrics, and generate PCoA plots for beta diversity analysis.

---

## Alpha Diversity Metrics

The following alpha diversity metrics were calculated:

* **Shannon’s Diversity Index**
  A quantitative measure of community richness

* **Observed Features**
  A qualitative measure of community richness

* **Faith’s Phylogenetic Diversity**
  A qualitative richness measure incorporating phylogenetic relationships

* **Pielou’s Evenness**
  A measure of community evenness

---

## Beta Diversity Metrics

The following beta diversity metrics were calculated:

* **Jaccard Distance**
  A qualitative measure of community dissimilarity

* **Bray-Curtis Distance**
  A quantitative measure of community dissimilarity

* **Unweighted UniFrac Distance**
  A qualitative phylogenetic dissimilarity measure

* **Weighted UniFrac Distance**
  A quantitative phylogenetic dissimilarity measure

---

## Choosing Sampling Depth

An important parameter for this analysis is:

```text id="r2k6v0"
--p-sampling-depth
```

This defines the even sampling (rarefaction) depth.

Since diversity metrics are sensitive to unequal sequencing depth across samples, QIIME2 randomly subsamples reads from each sample to the specified depth.

In this analysis:

```text id="0x1r5z"
--p-sampling-depth 10000
```

was selected based on the sequence distribution observed in `table.qzv`.

Samples containing fewer than 10,000 sequences were excluded from downstream diversity analysis.

---

## Run Core Diversity Analysis

```bash id="y7v6zt"
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 10000 \
  --m-metadata-file metadata.tsv \
  --output-dir core-metrics-results
```

---

## Output Artifacts

This command generates multiple QIIME2 artifacts including:

```text id="jlwm7g"
faith_pd_vector.qza
shannon_vector.qza
observed_features_vector.qza
evenness_vector.qza
jaccard_distance_matrix.qza
bray_curtis_distance_matrix.qza
unweighted_unifrac_distance_matrix.qza
weighted_unifrac_distance_matrix.qza
jaccard_pcoa_results.qza
bray_curtis_pcoa_results.qza
unweighted_unifrac_pcoa_results.qza
weighted_unifrac_pcoa_results.qza
rarefied_table.qza
```

---

## Output Visualizations

Interactive Emperor PCoA plots are also generated:

```text id="2prz3f"
jaccard_emperor.qzv
bray_curtis_emperor.qzv
unweighted_unifrac_emperor.qzv
weighted_unifrac_emperor.qzv
```


### 6. Alpha Rarefaction Analysis

Alpha rarefaction analysis was performed using the `qiime diversity alpha-rarefaction` visualizer to evaluate alpha diversity as a function of sequencing depth.

This analysis helps determine whether the sequencing depth was sufficient to capture the microbial richness present in the samples.

The visualizer computes one or more alpha diversity metrics at multiple sampling depths, ranging from low depth values up to the specified maximum depth.

At each depth:

* Multiple rarefied tables are generated
* Diversity metrics are calculated for all samples
* Average diversity values are plotted for comparison

Sample grouping based on metadata can also be visualized when a metadata file is provided.

---

## Run Alpha Rarefaction Analysis

```bash id="1yxb4u"
qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 10000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv
```

---

## Output Visualization

```text id="syf8fw"
alpha-rarefaction.qzv
```

---

## Interpretation of the Rarefaction Plot

The visualization contains two important plots:

## a. Top Plot - Alpha Rarefaction Curve

This plot is primarily used to determine whether the richness of the samples has been fully observed.

If the curves begin to **level out** (approach a slope of zero), this suggests that additional sequencing would likely not reveal many new features.

If the curves do **not level out**, it may indicate:

* Insufficient sequencing depth
* Remaining sequencing errors being interpreted as false diversity

This plot helps validate whether the chosen sampling depth is biologically meaningful.

---

## b. Bottom Plot - Sample Retention by Depth

This plot shows how many samples remain available at each rarefaction depth.

If the sampling depth exceeds the total sequence count of a sample, that sample is excluded from diversity calculations.

This is especially important when comparing groups using metadata because:

* Too high a sampling depth may remove many samples
* Too few remaining samples can make diversity comparisons unreliable

Therefore, both plots should be evaluated together when selecting the optimal rarefaction depth.


### 7 . Taxonomic Classification

To explore the taxonomic composition of the samples, taxonomy was assigned to the representative sequences (`FeatureData[Sequence]`) generated after DADA2 denoising.

Taxonomic classification was performed using the `q2-feature-classifier` plugin with a pre-trained **Naive Bayes classifier**.

The classifier used in this project was trained on:

**Greengenes2 2024.09 full-length reference sequences**

This classifier was applied to the representative sequences to identify the taxonomic composition of the microbial communities across samples.

---

## Download Pre-trained Classifier

Download the classifier using:

```bash id="bq0w6x"
wget https://data.qiime2.org/classifiers/sklearn-1.4.2/greengenes2/2024.09.backbone.full-length.nb.sklearn-1.4.2.qza
```

You may rename the downloaded file if needed for easier usage.

Example:

```text id="mzok0v"
2024.09.backbone.full-length.nb.sklearn-1.4.2.qza
```

---

## Assign Taxonomy

Run the following command:

```bash id="g4u5py"
qiime feature-classifier classify-sklearn \
  --i-classifier 2024.09.backbone.full-length.nb.sklearn-1.4.2.qza \
  --i-reads representative-sequences.qza \
  --o-classification taxonomy.qza
```

This will generate:

```text id="u6vmh2"
taxonomy.qza
```

---

## Visualize Taxonomic Classification

To generate a visualization of the taxonomy assignments:

```bash id="c2i6xj"
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```

This will generate:

```text id="5j2jhg"
taxonomy.qzv
```

The `taxonomy.qzv` file provides a detailed mapping of feature IDs to their assigned taxonomy and can be viewed using QIIME2 View.


### 8. Taxonomic Composition Visualization using Bar Plots

After assigning taxonomy to the representative sequences, the taxonomic composition of the samples can be explored using interactive taxonomy bar plots.

These bar plots provide a visual representation of the relative abundance of microbial taxa across all samples and help compare microbial community composition between different sample groups.

The visualization allows taxonomic exploration at multiple levels, including:

* Phylum
* Class
* Order
* Family
* Genus
* Species (when available)

This makes it easier to identify dominant taxa and observe differences across environmental samples.

---

## Generate Taxonomy Bar Plots

Run the following command:

```bash id="8w8w0e"
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```

This will generate:

```text id="7l5z5v"
taxa-bar-plots.qzv
```

The resulting visualization can be viewed using QIIME2 View and allows interactive exploration of taxonomic abundance across samples.


---

## Repository Structure

```text
QIIME2-Analysis/
│
├── README.md
├── environment.yml
├── Metadata/
├── QIIME2-Amplicon-2024.10 Installation.md
├── Results/
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
