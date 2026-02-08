# clinicalClassifyNMIBC
A tool aimed at improving the risk stratification of recurrence in Ta low grade bladder cancer by integrating clinical and transcriptomic data.

# BIOF520 Assignment 2: TaLG NMIBC Recurrence Risk Classifier

This repository contains code to a classifier that aims to improve recurrence risk stratification in Ta low-grade (TaLG) non–muscle invasive bladder cancer (NMIBC) using the UROMOL cohort for training/testing and the Knowles cohort for external validation.

The implementation is an integrative clinical–transcriptomic model using:
- Clinical variables (one-hot encoded; trained on UROMOL training data)
- Transcriptomic features (DE genes selected on UROMOL training data)
- Elastic net and LASSO (glmnet) for modeling and compact feature selection
- Used classifyNMIBC as a baseline model for performance evaluation


## Key Design  
- Data partition on the UROMOL dataset (75 training /25 test)
- Preprocessing based on UROMOL training data which includes:
  - clinical imputation
  - dummy encoding
  - expression scaling (center/scale)
  - DE gene selection (limma)
  - threshold selection (ROC Youden based ni training data)
- To avoid leakage, the Knowles dataset has been used for external validation and is not used for training or tuning.


---

## Data Requirements
This repo expects two `.rds` files (not committed to GitHub if restricted):
- `UROMOL_TaLG.teachingcohort.rds`
- `knowles_matched_TaLG_final.rds`

Update file paths in classifier.Rmd (rds files could not be uploded on github due to size):

uromol_path  <- "PATH/TO/UROMOL_TaLG.teachingcohort.rds"
knowles_path <- "PATH/TO/knowles_matched_TaLG_final.rds"

## Installation
Make sure to install the following packages before running the Rmd files.
install.packages(c("caret","glmnet","pROC","survival","dplyr","gridExtra"))
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager")
BiocManager::install("limma")

Install classifyNMIBC for baseline model:
devtools::install_github("sialindskrog/classifyNMIBC", build_vignettes = FALSE, upgrade = "never")

## Running Sequence
Run classifier.Rmd first. Don't clear the objects saved. Then run classifier.visualizations.Rmd to create visualizations.

## Repository Structure
- data 
- code
  - classifier.Rmd
  - classifier.visualizations.Rmd
- figures (folder)

## Workflow

```mermaid
flowchart TD
  A[Load the data\nUROMOL (RNA-seq) + Knowles (microarray)] --> B[Define the prediction task\nRecurrence (0/1) → NoRec vs. Rec]
  B --> C[Split UROMOL only\nStratified 75% train / 25% test\n]
  C --> D[Clean + encode clinical features (learned on training)\n- fill missing numeric with training medians\n- categories → factors (+ NA)\n- one-hot encode]
  C --> E[Clean + scale expression (learned on training)\n- fill missing gene values with training means\n- center/scale using traininf stats]
  D --> F[Make TEST + Knowles clinical compatible\n- unseen → NA\n- add missing columns as zeros\n- drop extra columns]
  E --> G[Make TEST + Knowles expression compatible\n- align gene space\n- fill missing genes\n- apply training scaling]
  E --> H[Pick candidate genes (training only)\nlimma DE → top K genes]
  H --> I[Build feature sets\nClinical-only / DE-only / Combined]
  I --> J[Train models on training\nElastic net (caret/glmnet)\nRepeated CV]
  I --> K[Build smaller interpretable model\nLASSO selects compact panel]
  J --> L[Choose decision threshold (training only)\nROC + Youden’s J]
  K --> L
  L --> M[Evaluate on UROMOL Test\nConfusion matrix + KM curves]
  L --> N[External validation on Knowles\nSame preprocessing + threshold]
  A --> O[Baseline: classifyNMIBC\nTraining threshold]
  O --> M
  O --> N

