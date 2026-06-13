# cfpb_consumer_banking_complaints

This repository contains our SIADS 696 Milestone II project on CFPB consumer banking complaints. The work combines structured complaint data, lightweight narrative features, supervised modeling, and evaluation focused on whether a complaint ultimately receives relief.

## Project Focus

The current end-to-end modeling path is centered on a binary relief target derived from `Company response to consumer`.

- Positive class: `Closed with monetary relief`
- Negative class: all other retained response outcomes in the final relief dataset

The repo also includes older notebook branches for earlier timely-response and broader relief experiments, but the `final` notebooks are the main deliverables for the current project direction.

## Repository Layout

- [notebooks](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks): active analysis, feature preparation, model development, and evaluation notebooks
- [old_notebooks](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/old_notebooks): archived notebook variants from earlier modeling directions
- [data/raw](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/data/raw): source extracts and raw complaint files
- [data/processed](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/data/processed): cleaned parquet files and model-ready datasets
- [visuals](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/visuals): charts exported from evaluation notebooks
- [output_tables](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/output_tables): small CSV outputs such as model comparison tables, ablation results, and failure examples
- [sample_data](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/sample_data): lightweight sample extracts that are safe to keep in Git for quick inspection

## Notebook Guide

The notebooks are designed to follow a fairly natural project flow from raw extraction through final evaluation.

### Data Preparation and Exploration

- [00_data_extraction.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/00_data_extraction.ipynb): builds the working complaint extracts from the raw CFPB data and writes project-specific parquet outputs
- [01_data_exploration.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/01_data_exploration.ipynb): structured data exploration, target inspection, and early data quality review
- [01a_data_exploration_narratives.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/01a_data_exploration_narratives.ipynb): narrative-focused exploration for missingness, text coverage, and qualitative review
- [02_narratives_cleaning.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/02_narratives_cleaning.ipynb): cleans complaint narratives and writes reusable text fields for downstream modeling
- [02b_narrative_sentiment.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/02b_narrative_sentiment.ipynb): runs VADER once on the cleaned narratives and writes sentiment-enriched parquet files so development and evaluation notebooks do not need to recompute sentiment every run

### Final Modeling Path

- [03_supervised_model_development_final.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/03_supervised_model_development_final.ipynb): main structured-feature supervised model development notebook for the final binary relief task
- [04_supervised_model_evaluation_final.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/04_supervised_model_evaluation_final.ipynb): evaluation notebook for the selected final model, including threshold analysis, learning-curve analysis, ablations, and failure analysis

### Narrative-Focused Final Path

- [03_supervised_model_development_final_narrative.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/03_supervised_model_development_final_narrative.ipynb): development notebook for the narrative-present subset, keeping the modeling path separate when text coverage matters
- [04_supervised_model_evaluation_final_narrative.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/04_supervised_model_evaluation_final_narrative.ipynb): evaluation notebook for the narrative-focused final model

## Data Notes

The primary final modeling dataset is:

- [consumer_banking_relief_vader.parquet](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/data/processed/consumer_banking_relief_vader.parquet)

This file is the cleaned, model-ready relief dataset with a precomputed VADER sentiment score. At the time of writing, it contains about `1.12M` rows and `20` columns.

Key fields include:

- complaint dates such as `Date received` and `Date sent to company`
- structured complaint descriptors such as `Product`, `Sub-product`, `Issue`, and `Sub-issue`
- company and submission context such as `Company` and `Submitted via`
- narrative fields including `Consumer complaint narrative`, `cleaned_consumer_narrative`, and `narrative_sentiment_score`
- outcome fields such as `Company response to consumer` and `Timely response?`
- the record identifier `Complaint ID`

Important modeling note:

- `narrative_sentiment_score` is now precomputed upstream so the final development and evaluation notebooks can load it directly instead of recalculating sentiment during every training run

## Recommended Run Order

For a clean rerun of the final project path, this is the intended order:

1. [00_data_extraction.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/00_data_extraction.ipynb)
2. [01_data_exploration.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/01_data_exploration.ipynb)
3. [01a_data_exploration_narratives.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/01a_data_exploration_narratives.ipynb)
4. [02_narratives_cleaning.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/02_narratives_cleaning.ipynb)
5. [02b_narrative_sentiment.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/02b_narrative_sentiment.ipynb)
6. [03_supervised_model_development_final.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/03_supervised_model_development_final.ipynb)
7. [04_supervised_model_evaluation_final.ipynb](/c:/Users/ntamm/OneDrive/Documents/1_MADS/1_Classes/696%20-%20Milestone%202/Project/cfpb_mortgage_complaints/notebooks/04_supervised_model_evaluation_final.ipynb)

If you want to compare the narrative-only path, run the two `final_narrative` notebooks after the sentiment-precompute step.

## Setup

Install dependencies with:

```bash
pip install -r requirements.txt
```

The current environment includes the core packages needed for data prep, notebook work, supervised modeling, class imbalance handling, and exported sentiment scoring.
