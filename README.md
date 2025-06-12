# PhyloerKit
- [1. Auto-MCMCTree.py]()

## 1. [Auto-MCMCTree.py](https://github.com/SunLab-MiaoPu/PhyloerKit/blob/main/src/Auto_MCMCTree.py)
**This script was designed for automating the MCMCTree (a software used for divergence time estimation) pipeline.**

### 🚀 [Manual MCMCTree steps](https://github.com/sabifo4/Tutorial_MCMCtree/tree/main) vs `Auto-MCMCTree.py`

| Manual MCMCTree Steps | This Script |
|-----------------------|-------------|
| 1. Manually convert FASTA to PHYLIP manually | ✅ Automatic conversion |
| 2. Manually calibrate the treefile manually(troublesome and time-consuming) | ✅ Automatically calibrate the tree  according to the user-provided calibration config file |
| 3. Manually create 3 control files | ✅ Automatically generate all CTL files for running MCMCTree according to the user-specified parameters|
| 4. Manually run `mcmctree` 1 time for preparation and 2 times for tree inference | ✅ Automatically handle all executions |
| 5. Manually execute convergence analysis | ✅ Automatically calculate R² to judgle the convergence |
| 6. Manually generate figures of convergence analysis and timetree with separate and additional tools | ✅ Automatic visualization |

**Typical manual workflow requires 10+ commands** → **This script reduces to ONE command to execute the full pipeline!!!**
**Thus, try to use this script to run MCMCTree automatically!!!**

## ⚙️ Requirements

- Python 3.6+
- MCMCTree (PAML package)
- BioPython
- pandas
- matplotlib
- rpy2 (for tree plotting)
- R package "MCMCtreeR"

Install dependencies:
```bash
conda create -n auto_mcmctree
conda activate auto_mcmctree
pip install biopython pandas matplotlib rpy2 tqdm
conda install bioconda::paml
R
install.packages("MCMCtreeR")
```

## 🔍 Pipeline overview

This Python script automates the complete MCMCTree workflow for Bayesian divergence time estimation. It handles:

- File format conversion (FASTA → PHYLIP)
- Tree calibration with fossil constraints
- MCMCTree execution (1st round for preparation and 2 rounds for timetree inference)
- Convergence diagnostics and results visualization
- Visualization of time-calibrated trees


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

## Test Run (with the [`example_data`](https://github.com/SunLab-MiaoPu/PhyloerKit/tree/main/example_dataset)):

#### Installation (with the directory `example_dataset`)
```
gitclone https://github.com/SunLab-MiaoPu/PhyloerKit
```

#### Step1: Change the working directory
```
cd ./PhyloerKit/
```

#### Step2: Run the script
We prepared two suites of input files (with different config files for calibration)

- Run the first type of input files
```bash
python ./src/Auto_MCMCTree.py \
-it ./example_dataset/Test-Auto-MCMCTree/input_ASTRAL_HybSuite_RLWP_sorted_rr.tre \
-ic ./example_dataset/Test-Auto-MCMCTree/input1_Elaeagnaceae.config \
-fd ./example_dataset/Test-Auto-MCMCTree/input1_alignments/ \
-o ./example_dataset/Test-Auto-MCMCTree/output1/ \
--plot_tree -p Eleagnus_1
```

- Run the second type of input files
```bash
python ./src/Auto_MCMCTree.py \
-it ./example_dataset/Test-Auto-MCMCTree/input_ASTRAL_HybSuite_RLWP_sorted_rr.tre \
-ic ./example_dataset/Test-Auto-MCMCTree/input2_Elaeagnaceae.config \
-fd ./example_dataset/Test-Auto-MCMCTree/input2_alignments/ \
-o ./example_dataset/Test-Auto-MCMCTree/output2/ \
--plot_tree -p Eleagnus_2
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
├── Elaeagnus_1.phy                    # Merged PHYLIP alignment as input file for running MCMCTree
├── Elaeagnus_1.calibrated.tre         # Calibrated Newick tree as input file for running MCMCTree
├── Elaeagnus_1.MCMCTree_dated.tre     # Final time tree generated via MCMCTree (NEXUS)
├── Elaeagnus_1.convergence_analysis.png  # The figure displaying the covergence analysis results
└── Elaeagnus_1.MCMCTree_dated.pdf     # Time tree visualization
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

## 📜 Full pipeline parameters

```
options:
  -h, --help            show this help message and exit
  -it, --input_tree INPUT_TREE
                        Input tree file (only Newick format)
  -ic, --input_config INPUT_CONFIG
                        Calibration points config file (.config format)
  -o, --output_dir OUTPUT_DIR
                        Output directory (default: .)
  -fd, --fasta_dir FASTA_DIR
                        The directory containing all alignments files (fasta format)
  -p, --prefix PREFIX   Prefix for the output file (default: mcmctree)
  -bi, --burn_in BURN_IN
                        Number of burnin for running MCMCTree (default: 200000)
  -sf, --sample_freq SAMPLE_FREQ
                        Number of sample frequency for running MCMCTree (default: 10)
  -ns, --num_samples NUM_SAMPLES
                        Number of samples for running MCMCTree (default: 50000)
  -rs, --recommended_sampling {small,medium,high,super}
                        The recommended sampling scheme for running MCMCTree.
  -ap, --age_precision AGE_PRECISION
                        Precision for the age values in the config file after dividing by 100 (default: 4)
  -ndata NDATA          Number of data for running MCMCTree (default: 5)
  -seqtype SEQTYPE      Sequence type for running MCMCTree (0: nucleotides; 1:codons; 2:AAs; default: 0)
  -clock CLOCK          Use clock for running MCMCTree (1: global clock; 2: independent rates; 3: correlated rates default: 2)
  -RootAge ROOTAGE      Age constraint on root age (used if no fossil for root, default: <1)
  -model MODEL          Model for running MCMCTree (0:JC69, 1:K80, 2:F81, 3:F84, 4:HKY85; default: 4)
  -cleandata CLEANDATA  Remove sites with ambiguity data (1:yes, 0:no; default: 0)
  -print PRINT          Print the output (0: no mcmc sample; 1: everything except branch rates; 2: everything; default: 1)
  -c, --colormap {viridis,plasma,inferno,magma,cividis,coolwarm,rainbow,jet,turbo}
                        Color scheme for the plot (default: viridis)
  -t, --tick_step TICK_STEP
                        Step size for colorbar ticks (default: 5)
  --plot_tree           Plot the tree (default: False)
  --no_ladderize_tree   Do not ladderize the tree when plotting the MCMCTree timetree (default: False)
```

## 📄 License

MIT License - Free for academic and commercial use
