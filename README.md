# Assignment 4: Neural Network Verification with α,β-CROWN

**Course**: Reliable and Trustworthy Artificial Intelligence  
**Topic**: Local robustness verification using α,β-CROWN (VNN-COMP winner 2021–2025)

---

## Overview

This project applies α,β-CROWN to verify the local $\ell_\infty$ robustness of
the same MNIST FC model used in Assignment #3, enabling a direct comparison
between SMT-based (Marabou) and bound-propagation-based (α,β-CROWN) verification.

**Verification query**: For 10 MNIST test images, verify that all inputs within
$\|\mathbf{x}' - \mathbf{x}\|_\infty \leq 0.01$ are classified as the same digit.

---

## Project Structure

```
.
├── alpha-beta-CROWN/        # α,β-CROWN verifier (git-ignored)
├── models/
│   └── mnist_fc.onnx        # Trained MNIST FC model (from Assignment #3)
├── logs/
│   ├── mnist_cnn_run.log    # Tutorial example log
│   └── mnist_fc_run.log     # Our verification log
├── config.yaml              # Verification configuration for our model
├── test.py                  # Entry point for grading
├── report.pdf               # Analysis report
├── requirements.txt
└── README.md
```

---

## Environment Setup

Python 3.11 with conda is recommended.

```bash
conda create -n alphabeta python=3.11 -y
conda activate alphabeta

# Clone α,β-CROWN (with submodules)
git clone --recursive https://github.com/Verified-Intelligence/alpha-beta-CROWN.git

# Install dependencies
cd alpha-beta-CROWN
pip install -e auto_LiRPA
conda env update --name alphabeta --file complete_verifier/environment_pyt280.yaml

# Fix graphviz version conflict
conda remove graphviz --force -y
pip install --no-cache-dir "graphviz==0.21" --force-reinstall
```

### MNIST Data Setup

The built-in data loader looks for MNIST in `complete_verifier/datasets/MNIST/raw/`.
Create a symbolic link to pre-downloaded MNIST files:

```bash
cd alpha-beta-CROWN/complete_verifier
rm -rf datasets/MNIST/raw
ln -s /path/to/your/mnist/data/MNIST/raw datasets/MNIST/raw
```

---

## How to Run

### Step 1: Verify installation with provided example

```bash
cd alpha-beta-CROWN/complete_verifier
python abcrown.py --config exp_configs/tutorial_examples/mnist_cnn_a_adv.yaml \
  2>&1 | tee ../../logs/mnist_cnn_run.log
```

### Step 2: Run verification on our MNIST FC model

```bash
cd alpha-beta-CROWN/complete_verifier
python abcrown.py --config ../../config.yaml \
  2>&1 | tee ../../logs/mnist_fc_run.log
```

### Step 3: Run the grading entry point

```bash
python3 test.py
```

---

## Verification Results

| Outcome | Count | Indices |
|---|---|---|
| safe-incomplete (verified) | 9 | 0,1,2,3,4,5,6,7,9 |
| unsafe-pgd (falsified) | 1 | 8 |
| timeout | 0 | — |

- **Verified accuracy**: 90% (ε=0.01, 10 samples)
- **Mean time**: 0.499s/sample, Max: 1.537s

---

## Installation Issues Encountered

1. **graphviz version conflict**: conda and pip installed conflicting versions,
   causing `AttributeError: module 'graphviz.backend' has no attribute 'ENCODING'`.
   Fixed by removing conda version and force-reinstalling via pip.

2. **MNIST data path**: `data_utils.py` looks for data in `datasets/MNIST/raw/`
   (not `data/`). Fixed by creating a symlink to pre-downloaded MNIST files.