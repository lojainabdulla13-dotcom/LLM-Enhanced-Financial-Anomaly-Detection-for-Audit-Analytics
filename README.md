# LLM-Enhanced Financial Anomaly Detection for Audit Analytics

Hybrid AI audit analytics system combining LLM-based analysis of unstructured audit reports with
Isolation Forest anomaly detection on structured NHS financial data.

## Overview

Auditors review large volumes of financial data and long narrative reports. This project combines two signals to flag NHS trusts that may be at risk of financial mismanagement:

1. **Unstructured (text):** an LLM reads each trust's annual audit report and gives it a
   **financial mismanagement score (0–1)**
2. **Structured (numbers):** an **Isolation Forest** flags trust-years whose costs and audit
   outcomes are unusual compared with the rest

## Dataset

`data/finaldatasetdissertation.xlsx` has **994 trust-year records** for NHS trusts from **2017-18 to 2021-22**. It was built from public sources:

| Source | Fields |
|---|---|
| NHS trust expenses | Expected cost, actual cost, variance, actual-to-expected ratio |
| National Audit Office auditor reports | Opinion on financial statements, value-for-money conclusion, additional reporting powers, link to the report |
| Organisation lookup | Trust name → org code, region, city, sector |
| LLM pipeline (notebook 02) | Financial mismanagement score (0–1) |
| Public reports / news | Confirmed fraud cases (where found) |

## Pipeline

| Notebook | What it does |
|---|---|
| `01_build_dataset.ipynb` | Merges the audit reports, org-code lookup and expenses data for each year |
| `02_llm_risk_scoring.ipynb` | Text features (regex + spaCy) → chunking → LLM chunk retrieval → summarisation → LLM mismanagement score |
| `03_isolation_forest.ipynb` | From-scratch Isolation Forest on costs and audit flags, adding an `iForest` anomaly flag |

### LLM scoring (notebook 02)

1. Clean the report text and extract features: money amounts, deficit/savings/overspend amounts, counts of risk, governance, KPI and negative terms, audit opinion type, and risk/negative term density
2. Split the report into overlapping 10,000-character chunks
3. **Gemini 2.0 Flash-Lite** picks the 10 chunks most relevant to financial mismanagement
4. Each chunk is summarised, then the summaries are combined into one executive summary
5. The LLM returns a 0–1 mismanagement score based on the summary and the extracted features

### Anomaly detection (notebook 03)

- Features: expected cost, actual cost, variance, actual-to-expected ratio, non-standard opinion, additional reporting powers exercised, qualified VfM conclusion
- 10 trees with a sample size of 256. `contamination = 0.315` flags about 31.5% of trust-years (313 of 994)

## Setup

```bash
pip install -r requirements.txt
python -m spacy download en_core_web_sm
export GEMINI_API_KEY="your-key"   # only needed for notebook 02
```

- Notebook 03 runs straight away on `data/finaldatasetdissertation.xlsx`
- Notebook 01 needs the raw source spreadsheets in `data/raw/<year>/`
- Notebook 02 needs the audit reports as text files in `data/reports_text/`, named like `R0A2017-2018.txt` (org code + year)

## Author

Lojain Abdulla
