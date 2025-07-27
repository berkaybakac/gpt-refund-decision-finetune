# OpenAI Fine-Tuning Project: Refund Decision Automation

This repository contains a complete pipeline for fine-tuning an OpenAI GPT-3.5 Turbo model to automate refund decisions based on structured call center data. The objective is to emulate the internal personnel's decision-making process — whether to **Accept** or **Reject** refund requests submitted by service providers.

---

## Directory Structure

```
├── 00_settings_check.ipynb         # API key, library checks, data load
├── 01_data_cleaning.ipynb          # Data cleaning, datetime parsing, label filtering
├── 02_feature_engineering.ipynb    # Time elapsed, transcript analysis, prompt creation
├── 03_filter_and_prompt.ipynb      # Final JSONL formatting for fine-tuning
├── 04_fine_tune_start.ipynb        # Fine-tuning initialization
├── 05_follow_job.ipynb             # Track fine-tuning job progress
├── 06_model_test.ipynb             # Run single example through the model
├── 07_eval_set_analysis.ipynb      # Evaluate the model with a test set
├── 08_eval_metrics.ipynb           # Compute precision, recall, F1-score
├── 09_error_analysis.ipynb         # Display misclassified examples
├── 10_export_eval_csv.ipynb        # Export evaluation results to CSV
└── fine_tuning_summary.md          # Project summary (delivery document)
```

---

## Process Summary

### Data Preparation

- Normalized Unicode and loaded as `.json`
- Filled NaNs, ensured consistent text types
- Calculated time elapsed (in minutes) between job assignment and refund request
- Extracted number of calls based on `||` separator in transcripts
- Filtered to include only `Personel Kabul Etti` and `Personel Redetti` labels

### Feature Engineering

- Added `Time_Elapsed_Minutes` as a new numeric feature
- Computed `HizmetVerenAramaSayisi` and `PartnerAramaSayisi` based on call transcript separators
- Constructed structured prompt with:
  - Reason for Refund Request
  - Call Transcripts (Provider and Partner)
  - Call Counts
  - Elapsed Time (Minutes)
- Generated supervised training examples in `prompt → completion` format

### Prompt Format (Training Data)

Each prompt uses a structured message format for consistent reasoning:

```text
Reason for Refund Request: …
Service Provider Call Transcripts: …
Service Provider Call Count: …
Partner Call Transcripts: …
Partner Call Count: …
Time Elapsed Between Job Submission and Refund Request: … minutes.
```

The model responds with a JSON object like:

```json
{"decision": "Accepted by Personnel", "reason": "müşteri yanlış adres vermiş"}
```

---

## Fine-Tuning & Testing

- Uploaded `fine_tuning_data.jsonl` to OpenAI via SDK
- Fine-tuned using `gpt-3.5-turbo-1106`
- Retrieved final model name after job completion
- Queried the model with sample and real data

---

## Evaluation

### Manual Test

- Ran a single hard-coded test to validate behavior

### Eval Set

- Selected 10 random examples from the dataset
- Compared expected vs predicted labels
- Computed accuracy score
- Printed failed cases with full JSON response

### Metrics

- Computed `accuracy`, `precision`, `recall`, `f1-score` using `scikit-learn`
- Analysis done only if prediction structure was valid JSON

### Error Analysis

- Displayed detailed reasoning of misclassified examples
- Helpful for understanding model weaknesses

### Export

- Saved evaluation results to `eval_output.csv` for reporting

---

## Deliverables

- `fine_tuning_data.jsonl` – Final training file
- `eval_output.csv` – Evaluation results (expected vs predicted vs response)
- Jupyter notebooks `00` → `10` – Complete project pipeline
- `fine_tuning_summary.md` – This report as standalone file

---

## Requirements

- Python ≥ 3.8
- `openai`, `pandas`, `scikit-learn`
- OpenAI API key (exported as `OPENAI_API_KEY`)

---

## Notes

- Project designed to reproduce personnel-level refund decision logic
- Prompt includes only visible features – model cannot hallucinate
- Evaluation can be extended to 100+ rows for more reliable metrics
