# CIMRL: Causality-Inspired Multimodal Representation Learning for Alzheimer's Disease Diagnosis

CIMRL is a research framework for robust Alzheimer's disease diagnosis using multimodal neuroimaging data. It integrates structural MRI-derived gray matter and white matter volumes with fMRI functional connectivity to learn disease-relevant representations that are less sensitive to domain shift, demographic confounders, and modality heterogeneity.

The method introduces a Causal Disentangled Variational Autoencoder (CD-VAE) to separate stable disease-related features from confounding factors, and an Uncertainty-aware Contrastive Fusion (UCF) module to adaptively fuse heterogeneous modalities under cross-modal alignment constraints.

This repository provides the core implementation used for multimodal representation learning, causal disentanglement, classification, feature extraction, and visualization.

---

## Key Features

- Multimodal Alzheimer's disease diagnosis with sMRI and fMRI data
- CD-VAE representation learning with Gaussian mixture priors
- Confounder-aware causal disentanglement using mutual information estimation
- Uncertainty-aware multimodal fusion for robust classification
- Utilities for latent-space visualization and functional connectivity analysis

---

## Repository Structure

```text
.
|-- requirements.txt
`-- cimrl_main/
    |-- mridata.py        # Dataset loading, MRI volume processing, and fMRI graph construction
    |-- gmvae.py          # CD-VAE for structural MRI representations
    |-- graphvae.py       # Graph CD-VAE for fMRI functional connectivity representations
    |-- train.py          # Main multimodal training and cross-validation script
    |-- get_pred.py       # Feature extraction and classifier evaluation utilities
    |-- regularizer.py    # Disease-progression order-preserving regularization
    |-- utils.py          # Mutual-information and entropy-based helper functions
    |-- mine.py           # Conditional MINE module
    |-- scatter.py        # PCA/KDE latent representation visualization
    `-- circos.py         # Functional connectivity circos visualization
```

---

## Installation

### 1. Create Environment

```bash
conda create -n cimrl python=3.9 -y
conda activate cimrl
```

### 2. Install PyTorch

Install a PyTorch version compatible with your CUDA driver. The original experiments used a CUDA-enabled PyTorch environment.

Example:

```bash
pip install torch torchvision torchaudio
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

Note: `requirements.txt` was exported from the original research environment. If any local `file://` or platform-specific wheel entries fail on a new machine, replace them with the corresponding package versions from PyPI or conda.

---

## Dataset Preparation

### Supported Datasets

- ADNI: Alzheimer's Disease Neuroimaging Initiative
- OASIS: Open Access Series of Imaging Studies

The datasets are not redistributed in this repository. Please obtain access from the official dataset providers and follow their data-use agreements.

### Expected Input Modalities

CIMRL uses three neuroimaging inputs:

- Gray matter volume from structural MRI
- White matter volume from structural MRI
- fMRI time-series data converted into a functional connectivity graph

### Expected Data Layout

The current dataset loader expects each subject to have fMRI time-series data and segmented MRI files. A recommended layout is:

```text
data/
`-- ADNI/
    |-- output/
    |   |-- fmri/
    |   |   |-- <SubjectID>.csv
    |   |   `-- ...
    |   `-- seg/
    |       |-- <SubjectID>/
    |       |   |-- <SubjectID>_pve_1.nii.gz
    |       |   `-- <SubjectID>_pve_2.nii.gz
    |       `-- ...
    `-- metadata.csv
```

Before running the code, update the dataset paths in the scripts:

```python
fmri_dir = "/path/to/ADNI/output/fmri"
mri_dir = "/path/to/ADNI/output/seg"
csv_file = "/path/to/ADNI/metadata.csv"
```

### Preprocessing

In the paper, structural MRI was preprocessed with SPM12 and CAT12, including noise filtering, bias field correction, skull stripping, tissue segmentation, spatial normalization, and gray/white matter extraction.

For fMRI, preprocessing was performed with FSL, including motion correction, slice timing correction, skull stripping, spatial normalization, temporal filtering, and smoothing. AAL116 atlas time series were extracted, and the first 90 non-cerebellar regions were retained for functional connectivity analysis.

---

## Training and Evaluation

Run commands from the `cimrl_main` directory:

```bash
cd cimrl_main
```

### 1. Train Structural MRI CD-VAE

```bash
python gmvae.py
```

This script trains the CD-VAE for structural MRI representations. It saves checkpoints and training metrics under the configured checkpoint directory.

### 2. Train fMRI Graph CD-VAE

```bash
python graphvae.py
```

This script trains the graph-based CD-VAE on fMRI functional connectivity features.

### 3. Train Multimodal Classifier

```bash
python train.py
```

The default training pipeline uses 5-fold stratified cross-validation and reports Accuracy, AUC, and Sensitivity. The model checkpoints and fold metrics are saved to the configured output directory.

### 4. Extract Features and Evaluate Classifiers

```bash
python get_pred.py
```

This utility can load pretrained encoders, extract latent representations, evaluate classifiers, and generate confusion matrices.

### 5. Generate Visualizations

```bash
python scatter.py
python circos.py
```

These scripts generate latent-space PCA/KDE visualizations and circos plots of functional connectivity patterns. They expect the required `.npy`, `.pt`, CSV, and checkpoint files to be available under the configured paths.

---

## Important Notes

- The scripts currently contain absolute paths from the original experimental environment. Please update all data, checkpoint, and output paths before running.
- Some analysis scripts expect local auxiliary files such as `checkpoints/`, `PRED/`, `ATT_RESULT/`, and atlas files.
- Do not upload ADNI or OASIS data to a public repository. Only release code, configuration examples, and instructions permitted by the dataset licenses.
- After the paper is accepted, we will upload all the code and model weights.

---

## License and Data Terms

This repository is intended for academic research use. Please add a project license file before public release. ADNI, OASIS, and other third-party datasets remain subject to their original licenses and data-use agreements.
