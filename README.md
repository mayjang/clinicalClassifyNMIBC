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


## Repository Structure
- data
  - UROMOL_TaLG.teachingcohort.rds
  - knowles_matched_TaLG_final.rds
- code
  - classifier.Rmd
  - classifier.visualizations.Rmd
- figures (folder)

## Workflow Diagram
