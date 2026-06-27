# cfpb_consumer_banking_complaints

This repository contains our SIADS 696 Milestone II project on CFPB consumer banking complaints. The current project setup is notebook-driven and the active workflow lives in the [notebooks](notebooks/) folder.

## Project Focus

The main modeling task is a binary classification problem based on `Company response to consumer`.

- Positive class: `Closed with monetary relief`
- Negative class: other retained outcomes in the filtered relief dataset

The active workflow now combines:

- structured complaint features
- cleaned complaint narratives
- unsupervised narrative features
- precomputed VADER sentiment
- supervised development and evaluation notebooks for the final relief task

## Repository Layout

- [notebooks](notebooks/): active project workflow and deliverables
- [old_notebooks](old_notebooks/): archived earlier notebook versions
- [data/raw](data/raw/): raw complaint extracts
- [data/processed](data/processed/): parquet outputs and saved modeling artifacts
- [output_tables](output_tables/): exported comparison tables and evaluation summaries
- [visuals](visuals/): exported plots from evaluation notebooks
- [sample_data](sample_data/): small Git-friendly samples for inspection

## Current Notebook Guide

### 1. Data Extraction and Exploration

- [00_data_extraction.ipynb](notebooks/00_data_extraction.ipynb): filters the CFPB complaint data down to the project scope and writes `consumer_banking_complaints.parquet`, `consumer_banking_monetary_complaints.parquet`, and `consumer_banking_relief.parquet`
- [01_data_exploration.ipynb](notebooks/01_data_exploration.ipynb): general exploration of the complaint data, response outcomes, and project scope
- [01a_data_exploration_narratives.ipynb](notebooks/01a_data_exploration_narratives.ipynb): exploration focused on narrative availability and text coverage

### 2. Narrative Preparation

- [02_narratives_cleaning.ipynb](notebooks/02_narratives_cleaning.ipynb): cleans complaint narratives and writes `data/processed/cleaned_complaints.parquet`
- [02b_narrative_sentiment.ipynb](notebooks/02b_narrative_sentiment.ipynb): computes VADER sentiment on the cleaned complaints data and writes `data/processed/cleaned_complaints_vader.parquet` and `data/processed/consumer_banking_relief_vader.parquet`

### 3. Unsupervised Narrative Feature Pipeline

- [05_unsupervised_feature_engineering.ipynb](notebooks/05_unsupervised_feature_engineering.ipynb): creates processed narrative text and writes `data/processed/processed_narratives.parquet`
- [06a_unsupervised_model_development_kmeans.ipynb](notebooks/06a_unsupervised_model_development_kmeans.ipynb): develops K-means narrative clusters and writes `data/processed/clustered_narratives.parquet`
- [06b_unsupervised_model_development_lda.ipynb](notebooks/06b_unsupervised_model_development_lda.ipynb): develops LDA topic features and writes `data/processed/lda_features.parquet`
- [07_unsupervised_join_features.ipynb](notebooks/07_unsupervised_join_features.ipynb): joins the unsupervised narrative features into `data/processed/final_unsupervised_features.parquet`
- [02c_narrative_sentiment_final_narrative.ipynb](notebooks/02c_narrative_sentiment_final_narrative.ipynb): adds VADER sentiment to the final unsupervised feature set and writes `data/processed/final_unsupervised_features_vader.parquet` and `data/processed/final_unsupervised_features_relief_vader.parquet`

### 4. Supervised Modeling and Evaluation

- [03a_supervised_baselines.ipynb](notebooks/03a_supervised_baselines.ipynb): builds simple baseline classifiers for the final relief task and writes `output_tables/03a_supervised_baseline_results.csv`
- [03_supervised_model_development_final_clean.ipynb](notebooks/03_supervised_model_development_final_clean.ipynb): main supervised development notebook for the final all-records relief model using `data/processed/final_unsupervised_features_relief_vader.parquet`
- [04_supervised_model_evaluation_final_clean.ipynb](notebooks/04_supervised_model_evaluation_final_clean.ipynb): evaluates the main final relief model and exports the `04_final_*_clean` tables and visuals
- [03_supervised_model_development_final_narrative.ipynb](notebooks/03_supervised_model_development_final_narrative.ipynb): supervised development notebook for the narrative-present subset where `narrative_present == 1`
- [04_supervised_model_evaluation_final_narrative.ipynb](notebooks/04_supervised_model_evaluation_final_narrative.ipynb): evaluates the narrative-only final model and exports the `04_final_narrative_*` tables and visuals

## Recommended Run Order

For a clean end-to-end rerun of the current project setup:

1. [00_data_extraction.ipynb](notebooks/00_data_extraction.ipynb)
2. [01_data_exploration.ipynb](notebooks/01_data_exploration.ipynb)
3. [01a_data_exploration_narratives.ipynb](notebooks/01a_data_exploration_narratives.ipynb)
4. [02_narratives_cleaning.ipynb](notebooks/02_narratives_cleaning.ipynb)
5. [02b_narrative_sentiment.ipynb](notebooks/02b_narrative_sentiment.ipynb)
6. [05_unsupervised_feature_engineering.ipynb](notebooks/05_unsupervised_feature_engineering.ipynb)
7. [06a_unsupervised_model_development_kmeans.ipynb](notebooks/06a_unsupervised_model_development_kmeans.ipynb)
8. [06b_unsupervised_model_development_lda.ipynb](notebooks/06b_unsupervised_model_development_lda.ipynb)
9. [07_unsupervised_join_features.ipynb](notebooks/07_unsupervised_join_features.ipynb)
10. [02c_narrative_sentiment_final_narrative.ipynb](notebooks/02c_narrative_sentiment_final_narrative.ipynb)
11. [03a_supervised_baselines.ipynb](notebooks/03a_supervised_baselines.ipynb)
12. [03_supervised_model_development_final_clean.ipynb](notebooks/03_supervised_model_development_final_clean.ipynb)
13. [04_supervised_model_evaluation_final_clean.ipynb](notebooks/04_supervised_model_evaluation_final_clean.ipynb)
14. [03_supervised_model_development_final_narrative.ipynb](notebooks/03_supervised_model_development_final_narrative.ipynb)
15. [04_supervised_model_evaluation_final_narrative.ipynb](notebooks/04_supervised_model_evaluation_final_narrative.ipynb)

If you only want the main supervised path, the minimum downstream chain is `00 -> 02 -> 05 -> 06a -> 06b -> 07 -> 02c -> 03a -> 03_supervised_model_development_final_clean -> 04_supervised_model_evaluation_final_clean`.

## Key Active Datasets

- `data/processed/consumer_banking_complaints.parquet`: filtered project-scope complaint data
- `data/processed/cleaned_complaints.parquet`: structured complaint data with cleaned narratives
- `data/processed/consumer_banking_relief_vader.parquet`: earlier relief dataset with sentiment
- `data/processed/final_unsupervised_features.parquet`: combined structured and unsupervised narrative features
- `data/processed/final_unsupervised_features_relief_vader.parquet`: current main modeling dataset for the supervised final notebooks

## Setup

Install dependencies with:

```bash
pip install -r requirements.txt
```

The current `requirements.txt` includes the notebook, data, modeling, and sentiment packages used by the workflow:

- `duckdb`
- `imbalanced-learn`
- `ipykernel`
- `ipython`
- `joblib`
- `jupyterlab`
- `matplotlib`
- `numpy`
- `pandas`
- `polars`
- `pyarrow`
- `requests`
- `scikit-learn`
- `seaborn`
- `vaderSentiment`

Then launch Jupyter and run the notebooks from the [notebooks](notebooks/) folder.
