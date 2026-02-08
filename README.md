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

```text
Input UROMOL + Knowles
            |
            v
 Output: Recurrence (NoRec vs Rec)
            |
            v
Split UROMOL (75% train / 25% test)
            |
            +---------------------------+
            |                           |
            v                           v
Clinical preprocessing (training)     Expression preprocessing (training)
- fill missing numeric (train medians)  - fill missing genes (train means)
- clean categories (+ NAs)          - center/scale using train stats
- one-hot encode                        |
            |                           |
            v                           v
Project Test data + Knowles clinical         Project Test data + Knowles expression
- align dummy columns                   - align gene space to UROMOL
- unseen → Unknown                      - fill missing genes
- add missing columns (0)               - apply train scaling
            |                           |
            +------------+--------------+
                         v
Feature selection (training): limma DE → top genes
                         |
                         v
Build feature sets (clinical categories / DE-only / combined)
                         |
                         v
Train models on Training data (elastic net, LASSO panel)
                         |
                         v
Choose threshold on Training data (ROC Youden)
                         |
                         v
Evaluate on UROMOL TEST + external Knowles for validation
- confusion matrices + metrics
- Kaplan–Meier curves
Compare to baseline (classifyNMIBC; threshold from triaining UROMOL data)

