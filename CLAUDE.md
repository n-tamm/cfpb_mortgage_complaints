# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a SIADS 696 Milestone II project applying supervised and unsupervised machine learning to CFPB (Consumer Financial Protection Bureau) consumer banking complaints. The goal is to predict complaint outcomes (monetary relief, non-monetary relief, timely response) and to discover latent themes in complaint narratives.

## Setup

```bash
pip install -r requirements.txt
```

The virtual environment is at `.venv/` (Python 3.x) and `.venv312/` (Python 3.12). Activate with `.venv\Scripts\activate` on Windows.

Launch notebooks:
```bash
jupyter lab
```

All notebooks live in `notebooks/` and use relative paths like `../data/...`. Run notebooks from the `notebooks/` directory.

## Data Pipeline — Notebook Execution Order

The notebooks follow a numbered sequence. Data files are not committed to git (only `.gitkeep` placeholders exist); run in order to regenerate:

| Notebook | Purpose | Input → Output |
|---|---|---|
| `00_data_extraction.ipynb` | Download CFPB CSV, convert to Parquet, filter to banking products (2016–2026) | Web → `data/raw/consumer_complaints.parquet`, `data/processed/consumer_banking_complaints.parquet`, `data/processed/consumer_banking_monetary_complaints.parquet` |
| `01_data_exploration.ipynb` | EDA on raw parquet | `data/raw/consumer_complaints.parquet` |
| `01a_data_exploration_narratives.ipynb` | EDA focused on complaint narratives | same |
| `02_narratives_cleaning.ipynb` | Clean raw narratives (REDACT PII, expand contractions, etc.) | → `data/processed/cleaned_complaints.parquet` |
| `03_supervised_model_development_*.ipynb` | Train/tune 4 model families (LR, KNN, RF, LinearSVC) for each target | → `data/processed/best_*_model.joblib`, holdout prediction parquets |
| `04_supervised_model_evaluation_*.ipynb` | Evaluate saved models on holdout | reads joblib + parquets |
| `05_unsupervised_feature_engineering.ipynb` | Preprocess narratives for clustering (lemmatize, remove stopwords) | `cleaned_complaints.parquet` → `data/processed/processed_narratives.parquet` |
| `06a_unsupervised_model_development_kmeans.ipynb` | TF-IDF + MiniBatchKMeans (k=18); saves model artifacts | `processed_narratives.parquet` → `data/processed/clustered_narratives.parquet` |
| `06b_unsupervised_model_development_lda.ipynb` | LDA topic modeling (20 topics) | `processed_narratives.parquet` → `data/processed/lda_features.parquet` |
| `07_unsupervised_join_features.ipynb` | Join K-Means clusters + LDA topic probabilities | → `data/processed/final_unsupervised_features.parquet` |

## Architecture

### Data

- **Source**: CFPB Consumer Complaint Database (`complaints.csv.zip` downloaded from `files.consumerfinance.gov`)
- **Products covered**: Mortgage, Checking/savings account, Credit card, Credit card or prepaid card, Bank account or service
- **Date range**: 2016-01-01 to 2026-05-31
- **Key columns**: `Product`, `Sub-product`, `Issue`, `Sub-issue`, `Consumer complaint narrative`, `Company`, `State`, `ZIP code`, `Company response to consumer`, `Timely response?`, `Complaint ID`

### Supervised Learning

Three binary classification tasks, each with its own notebook pair (development + evaluation):
- **Monetary relief**: `Company response to consumer == "Closed with monetary relief"` (balanced ~57/43)
- **Non-monetary relief**: similar binary framing
- **Timely response**: `Timely response? == True` (imbalanced; uses SMOTE)

Each development notebook:
1. Leakage-aware feature engineering (excludes `Date sent to company`, `Company public response`, `Timely response?` for monetary/relief targets)
2. Structured features: calendar features from `Date received`, narrative presence/length indicators, top-N bucketing of high-cardinality categoricals (`Company`, `Issue`, `Sub-product`, `zip3`)
3. Pipelines with `ColumnTransformer` + `GridSearchCV` (refit on F1) for 4 model families
4. `QUICK_TEST_MODE = True` flag in notebooks — reduces grid size and CV folds for speed; set to `False` for full runs

Model artifacts saved to `data/processed/` as `.joblib` files; these are git-ignored.

### Unsupervised Learning

- **Text preprocessing**: lowercase → expand contractions → tokenize (alpha only) → remove stopwords + custom (`redacted`, `redacted_date`) → lemmatize (noun-mode)
- **TF-IDF**: `max_features=20000`, `ngram_range=(1,2)`, `min_df=10`, `max_df=0.95`
- **K-Means**: `MiniBatchKMeans` with k=18, `batch_size=1000`. Artifacts (`tfidf_vectorizer.joblib`, `kmeans_clusters.joblib`) are saved in `notebooks/` (git-ignored). When re-running notebook 06a, load the saved vectorizer/model and call `.transform()` / `.fit_predict()` rather than re-fitting to avoid drift.
- **LDA**: 20-topic LDA producing `dominant_topic` + per-topic probability columns (`topic_0_prob` … `topic_19_prob`)
- **Final features**: K-Means `cluster` + `distance_to_centroid` joined with LDA topic probs on `Complaint ID`

### Key Design Choices

- **Polars** is used for large-scale data loading/filtering (`pl.scan_parquet`); **Pandas** is used within model notebooks after filtering to modeling subsets
- **DuckDB** is used for SQL-style joins and format conversions (CSV → Parquet, joining unsupervised features)
- Narrative features in supervised notebooks are intentionally kept simple (presence flag, char/word count) to isolate the contribution of unsupervised-generated features when comparing models
- `PROJECT_ROOT` is inferred at runtime: `Path.cwd().resolve().parent if Path.cwd().name == "notebooks" else Path.cwd().resolve()`