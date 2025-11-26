# Antibody Developability Prediction Using CNNs

This project builds a sequence-based classifier to predicts antbody developability. Positive examples come from Thera-SAbDab, and negative (unlabeled) examples come from the Observed Antibody Space (OAS). A lightweight 1D CNN is trained on VH amino-acid sequences after careful preprocessing, masking, and stratified splitting, and CDR3-grouping to avoid clone leakage.


---

## Overview

Developability issues (aggregation, stability, expression, manufacturability) are major bottlenecks in therapeutic antibody discovery. This project investigates whether VH-sequence-only features can help distinguish clinically advanced antibodies from typical human memory B-cell repertoire sequences. Since the background class is unlabeled and may contain developable antibodies, the task should be interpreted as positive-vs-background discrimination, not as identifying absolute non-developable antibodies. The workflow includes:

- Curating **182 therapeutic VH sequences** from Thera-SAbDab
- Filtering and downsampling **2,000 OAS VH sequences** to match IGHV family distributions
- Masking the first 8 N-terminal residues to avoid dataset-specific shortcuts
- Grouping by CDR3 to prevent clone leakage across splits
- Training a small multi-kernel 1D CNN classifier
- Evaluating performance using AUROC and AUPRC under strong class imbalance

---

## Data

### Thera-SAbDab (Positive Set)

Therapeutic antibodies were filtered from Thera-SAbDab by:

- `Format == "whole mab"`
- `Highest_Clin_Trial (Feb '25)` ∈ {approved, phase-iii}
- `Est. Status == "active"`
- Genetics ∈ {genetically human, humanised}

Only VH sequences were used. After removing VH duplicates, **182 unique VH sequences** remained.

Each VH sequence was processed with **ANARCI** (IMGT scheme) to extract FR1–FR4, CDR1–CDR3, and IGHV family assignments. IGHV family distributions were used to match the OAS downsampling.

---

## OAS (Unlabeled Repertoire Set)

OAS sequences were filtered on the website using:

- Species: human
- BSource: PBMC
- BType: Memory B cells
- Chain: Heavy
- Type: IGHG
- Disease: None
- Vaccine: None

This returned **444,594 unique sequences** from 8 studies.

Processing:

- Kept canonical amino acids only
- Kept VH sequences of length **90–150 aa**
- Removed duplicates
- Assigned IGHV families
- Restricted to families overlapping with the Thera-SAbDab set
- **Downsampled to 2,000 sequences**, matching IGHV family distributions

This forms the **background/unlabeled** class (label = 0).

---

## Preprocessing & Splitting

- All sequences were tokenized into indices over:
  - 20 canonical amino acids + `X` (mask token)
- First **8 amino acids were masked to `X`** in all sequences to prevent trivial therapeutic-specific motifs from dominating.
- Train/validation/test splits were generated using **CDR3-based grouping** to prevent clone leakage.
- Stratified splitting ensured balanced representation across folds.

---

## Model

A lightweight **multi-kernel 1D Convolutional Neural Network (CNN)** was used:

- Embedding dimension: **64**
- Convolutional filters: **128** per kernel size
- Kernel sizes: **3, 5, 7**
- Global max pooling over sequence length
- Dropout: **0.3**
- Fully connected output layer → binary logit

The model processes padded batches of tokenized VH sequences.

---

## Training

- Loss: **BCEWithLogitsLoss** with `pos_weight ≈ 10.7` (to correct strong class imbalance)
- Optimizer: **AdamW**
  - Learning rate: `1e-3`
  - Weight decay: `1e-4`
- Batch size: **32**
- Device: GPU if available

Example training progression (summarized):

- Training accuracy increased steadily
- Validation accuracy peaked around **0.92**
- Loss curves showed stable learning

---

## Evaluation

The test set is highly imbalanced:

- 295 background (0)
- 25 positive (1)
- Majority baseline accuracy: **0.922**

More informative metrics were computed:

**Test results:**

- **AUROC: 0.9833**
- **AUPRC: 0.8502**

The model substantially outperforms the majority-class baseline and demonstrates strong ranking capability.

---

## Repository Structure

``├── notebooks/``

``│ ├── 01_therasabdab_preprocessing.ipynb``

``│ ├── 02_oas_preprocessing.ipynb``

``│ └── 03_cnn_training.ipynb``

``│``

``├── models/``

``│ └── model_00.pth``

``│``

``├── data/``

``│└── README.md``

``├── README.md``

``├── LICENSE``

``└── .gitignore``



## How to Run

1. Clone the repository:

bash
git clone https://github.com/<your-username>/antibody-developability-prediction.git
cd antibody-developability-prediction

2. Install dependencies:

bash
pip install -r requirements.txt

Download Thera-SAbDab and OAS filtered datasets and place them in data/raw/.

Run the notebooks in order for preprocessing and training.



## Future Work

Explore transformer-based embeddings (ESM, ProtBERT)

Use positive–unlabeled (PU) learning methods

Add multi-task developability labels (aggregation, stability)

Compare VH-only models with paired VH–VL models



## References

**Thera-SAbDab:** Therapeutic Structural Antibody Database

**OAS: Observed Antibody Space**

**ANARCI:** Antigen Receptor Numbering and Classification



## Repository Structure

