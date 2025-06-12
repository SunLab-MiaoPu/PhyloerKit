# PhyloerKit
- [Auto-MCMCTree.py]()

## Auto-MCMCTree.py (MCMCTree Automation Pipeline)
**A one-command solution for divergence time estimation with MCMCTree**

## 🔍 Overview

This Python script automates the complete MCMCTree workflow for Bayesian divergence time estimation. It handles:

- File format conversion (FASTA → PHYLIP)
- Tree calibration with fossil constraints
- MCMCTree execution (1st round for preparation and 2 rounds for timetree inference)
- Convergence diagnostics and results visualization
- Visualization of time-calibrated trees

### 🚀 Key Advantage vs Manual Operation

| Manual MCMCTree Steps | This Script |
|-----------------------|-------------|
| 1. Convert FASTA to PHYLIP manually | ✅ Automatic conversion |
| 2. Calibrate the treefile manually(troublesome and time-consuming) | ✅ Automatically calibrate the tree  according to the user-provided calibration config file |
| 3. Create 3 control files manually | ✅ Automatically generate all CTL files for running MCMCTree according to the user-specified parameters|
| 4. Run `mcmctree` 1 time for preparation and 2 times for tree inference | ✅ Automatically handle all executions |
| 5. Manually compare output files for convergence | ✅ Automatically calculate R² to judgle the convergence |
| 6. Generate figures with separate tools | ✅ Built-in visualization |

**Typical manual workflow requires 10+ commands** → **This script reduces to ONE command**

## ⚙️ Requirements

- Python 3.6+
- MCMCTree (PAML package)
- BioPython
- pandas
- matplotlib
- rpy2 (for tree plotting)
- R package "mcmctreeR"

Install dependencies:
```bash
pip install biopython pandas matplotlib rpy2 tqdm
```

## 📝 Input Files

> [!TIP] Only three input files are needed for running this script!

1. **Input Tree** (`-it`):
   - Newick format
   - Tip names must match FASTA headers and calibration file

2. **Calibration Config** (`-ic`):
   ```text
   mrca = Human Chimpanzee
   min = 6.5
   max = 10.0
   
   mrca = Mammal Bird
   min = 160.0
   ```

3. **FASTA Directory** (`-fd`):
   - Contains one FASTA per locus
   - All sequences must be aligned
   - Example:
     ```
     >Human
     ATGC...
     >Chimpanzee
     ATGC...
     ```


## 🏃‍♂️ Basic Usage:
```bash
python Auto-MCMCTree.py \
    -it input.tre \
    -ic calibrations.config \
    -fd fasta_directory/ \
    -o results/
```

### Test Run (with example data):
```bash
# Download example dataset


# Execute pipeline
python mcmctree_auto.py \
    -it test_data/species.tre \
    -ic test_data/calibrations.config \
    -fd test_data/alignments/ \
    -p test_run \
    --plot_tree
```


## 🛠️ Core Parameters

### Execution Control:
```bash
-bi 200000      # Burn-in (default: 200000)
-sf 10          # Sampling frequency (default: 10)
-ns 50000       # Number of samples (default: 50000)
```

### Model Settings:
```bash
-model 4        # Substitution model (0-4, default: 4=HKY85)
-clock 2        # Clock model (1-3, default: 2=independent rates)
-seqtype 0      # 0=nucleotides, 1=codons, 2=AAs
```

## 📊 Output Files

```
results/
├── mcmctree.phy                    # Merged PHYLIP alignment
├── mcmctree.calibrated.tre         # Calibrated Newick tree
├── mcmctree.MCMCTree_dated.tre     # Final time tree (NEXUS)
├── mcmctree.convergence_analysis.png  # MCMC diagnostics
└── mcmctree.MCMCTree_dated.pdf     # Time tree visualization
```

## 💡 Advanced Options

### Recommended Sampling Schemes:
```bash
-rs small    # Quick test (bi=10000, ns=10000)
-rs medium   # Medium (bi=100000, ns=20000)
-rs high     # Production (bi=200000, ns=50000)
```

### Visualization Control:
```bash
--plot_tree          # Generate PDF tree visualization
--no_ladderize_tree  # Disable automatic ladderizing
-c viridis           # Color scheme for plots
-t 10                # Node label frequency on colorbar
```

## ⚠️ Troubleshooting

**Common Issues:**
1. *"Species not found in tree"*:
   - Verify tip names match exactly between tree, FASTA, and config

2. *Low R² in convergence plot*:
   - Increase burn-in and sample size:
     ```bash
     -bi 500000 -ns 100000
     ```

3. *Empty output directory*:
   - Script cleans the directory by default - backup previous results

## 📜 Full Parameter Reference

| Parameter        | Type  | Default | Description |
|------------------|-------|---------|-------------|
| `-it`, `--input_tree` | file | Required | Input Newick tree |
| `-ic`, `--input_config` | file | Required | Calibration config |
| `-fd`, `--fasta_dir` | dir | Required | FASTA alignments |
| `-o`, `--output_dir` | str | . | Output directory |
| `-bi`, `--burn_in` | int | 200000 | MCMC burn-in |
| `-model` | int | 4 | Substitution model |
| `-RootAge` | str | '<1' | Root age constraint |

## 📄 License

MIT License - Free for academic and commercial use
