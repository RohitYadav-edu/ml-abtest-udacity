# ml-abtest-udacity
End-to-End A/B testing and uplift modeling on the Udacity dataset with statistical analysis, CUPED, and AWS deployment.

# Udacity A/B Test Analysis and Uplift Modeling

## Overview
This project analyzes the Udacity A/B Test dataset to understand user behavior and estimate the causal impact of experiments using:
- A/B testing (z-test, p-values, confidence intervals)
- CUPED variance reduction
- Uplift modeling (T-learner, LightGBM)
- Simple AWS Lambda API for treatment assignment

## Folder Structure
ml-abtest-udacity/

├── data/ # Raw & cleaned data (not pushed to GitHub)

├── notebooks/ # Jupyter notebooks for EDA and experiments

├── src/ # Python scripts (data cleaning, modeling)

├── models/ # Saved models

├── api/ # AWS Lambda / FastAPI code

├── tests/ # Unit tests

└── docs/ # Reports, diagrams


## Next Steps
1. Load and explore the dataset.
2. Clean and prepare data.
3. Run power analysis & A/B test.
4. Apply CUPED.
5. Train uplift models.
6. Deploy a small AWS Lambda.
