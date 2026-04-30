## Environment Setup

Two options are provided to set up the QIIME2 environment for this project.

---

### Option 1: Exact Reproducibility (Recommended)

This option recreates the exact software environment used in this analysis, ensuring consistent and reproducible results.

#### Create the Environment

```bash id="r6m1xv"
conda env create -f environment_exact.yml
```

#### Activate the Environment

```bash id="w0p3jf"
conda activate qiime2-2024.2
```

This method guarantees that all package versions match those used in the original workflow.

---

### Option 2: Manual Installation of QIIME2

Alternatively, you can install QIIME2 manually using the official distribution.

#### Install QIIME2 Amplicon Distribution

```bash id="k9y2sd"
conda env create -n qiime2-amplicon-2024.10 \
  --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
```

#### Activate the Environment

```bash id="h3x8pt"
conda activate qiime2-amplicon-2024.10
```

---

**Note:**
The manual installation may result in minor version differences compared to the exact environment used in this project. For fully reproducible results, Option 1 is recommended.

### Activate and Verify QIIME2 Installation

After creating the QIIME2 environment, activate it using:

```bash id="v7p9ks"
conda activate qiime2-amplicon-2024.10
```

To deactivate the environment when finished:

```bash id="x3n2qf"
conda deactivate
```

---

### Test the Installation

To verify that QIIME2 has been installed correctly, run:

```bash id="p8d4mj"
qiime --help
```

If the command executes without errors and displays the QIIME2 help menu, the installation was successful.
