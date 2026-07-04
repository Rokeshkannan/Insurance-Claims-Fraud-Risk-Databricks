# Insurance-Claims-Fraud-Risk-Databricks
End-to-end insurance claims cost &amp; fraud risk analysis using Databricks medallion architecture, Spark SQL, and AI/BI Dashboards
# Insurance Claims Cost & Fraud Risk Analysis

## Overview

An end-to-end analytics pipeline built on Databricks that analyzes auto insurance claims data to surface cost drivers and flag high-risk claims for fraud investigation. Built using medallion architecture (Bronze → Silver → Gold), Spark SQL, and a rules-based fraud risk scoring engine, with results visualized in a Databricks AI/BI Dashboard.

**Why this project:** Insurance claims teams lose money two ways — paying out inflated or fraudulent claims, and failing to understand which segments drive the highest cost. This project simulates the kind of analysis a Claims/Risk Analyst would be asked to deliver: which claims need investigation, and which customer segments are the most expensive.

## Architecture

```
Bronze (raw ingestion)
   ↓
Silver (cleaned, deduplicated, feature-engineered)
   ↓
Gold (business aggregates + fraud risk scores)
   ↓
Databricks AI/BI Dashboard
```

**Tech stack:** Databricks (Community Edition), Delta Lake, Spark SQL, Databricks AI/BI Dashboards, Unity Catalog Volumes

## Dataset

Auto Insurance Claims dataset (Kaggle) — 1,000 claims across 3 US states (OH, IN, IL), with policy details, incident details, and a `fraud_reported` ground-truth label used to validate the risk scoring logic.

- Base fraud rate in dataset: **24.7%**

## Pipeline Details

**Bronze layer:** Raw CSV ingested as-is into a Delta table, no transformations — preserves the original data for traceability.

**Silver layer:** 
- Standardized missing-value markers (`"?"` → `"Unknown"`) across `collision_type`, `property_damage`, `police_report_available`
- Engineered features: `claim_to_premium_ratio`, `pct_vehicle_claim`, `pct_injury_claim`

**Gold layer:**
- `gold_loss_ratio_by_state` — loss ratio and fraud rate by policy state
- `gold_loss_ratio_by_age` — loss ratio by customer age band
- `gold_claims_trend` — monthly claim volume and cost
- `gold_severity_analysis` — fraud rate by incident type/severity
- `gold_fraud_risk_scored` — rules-based fraud risk score and bucket (High/Medium/Low) per claim

## Fraud Risk Scoring Methodology

A rules-based score (not ML) was chosen for transparency and defensibility — this mirrors how many real claims teams operate before graduating to ML-based fraud models. Points are assigned based on:

| Signal | Rule | Points |
|---|---|---|
| Customer tenure | < 12 months as customer | +2 |
| Claim-to-premium ratio | > 90th percentile (74.69) | +2 |
| Claim-to-premium ratio | > 75th percentile (58.69) | +1 |
| Incident severity | Major Damage or Total Loss | +1 |
| Police report | Not on file / unknown | +1 |
| Witnesses | Zero witnesses despite known collision type | +1 |

Thresholds for the claim-to-premium ratio were derived from the actual percentile distribution of the dataset, not arbitrary cutoffs.

**Validation against ground truth (`fraud_reported` label):**

| Risk Bucket | Total Claims | Actual Frauds | Fraud Capture Rate |
|---|---|---|---|
| High | 94 | 33 | 35.1% |
| Medium | 468 | 145 | 31.0% |
| Low | 438 | 69 | 15.8% |

**Key finding:** Claims flagged High or Medium risk (562 of 1,000) were caught at **1.3–1.4x the baseline fraud rate**, while the Low-risk bucket cut fraud incidence by more than a third relative to baseline. This demonstrates that even simple, interpretable business rules can meaningfully prioritize investigation effort — a realistic first step before layering in machine learning.

## Dashboard

Built natively in Databricks AI/BI Dashboards, connected directly to the Gold layer tables via SQL datasets (no manual export required).

**Includes:**
- KPI summary: total policies, total claim amount, overall fraud rate
- Claims by risk bucket (donut)
- Loss ratio by state
- Fraud capture rate by risk bucket
- Monthly claims cost trend
- Fraud rate by incident severity
- Loss ratio by age band
- High-risk claims table (drill-down list for investigators)

<img width="950" height="476" alt="Screenshot 2026-07-04 103713" src="https://github.com/user-attachments/assets/2ba11e55-631f-4004-93e3-adb7998c0496" />
<img width="944" height="467" alt="Screenshot 2026-07-04 103843" src="https://github.com/user-attachments/assets/c6f633ca-22a4-41b3-b7b4-fe7f031220d9" />
<img width="938" height="473" alt="Screenshot 2026-07-04 103905" src="https://github.com/user-attachments/assets/1d1ed60e-eae0-46a5-9f6f-c646eca7d551" />
<img width="944" height="475" alt="Screenshot 2026-07-04 103926" src="https://github.com/user-attachments/assets/5f4f61f5-367c-4695-bf0a-a67e270aeb06" />


## Key Business Insights

- Illinois (IL) had the highest loss ratio among the three states in the dataset, indicating disproportionate claims cost relative to premiums collected.
- Customers aged 60+ showed the highest loss ratio (50.28) across age bands.
- Major Damage incidents carried a substantially higher fraud rate than Minor Damage or Trivial Damage claims.
- January and February 2015 accounted for the vast majority of claims cost in the dataset window.

## What I'd Improve With More Time

- Layer in an ML classifier (e.g., logistic regression or XGBoost) trained on the same features, and compare its precision/recall against this rules-based baseline
- Add day-level or report-lag features if a "claim filed date" field were available, since time-to-report is a well-known fraud signal not present in this dataset
- Automate ingestion via a scheduled job rather than manual upload, for a closer-to-production pattern

## Links

- GitHub: https://github.com/Rokeshkannan
- LinkedIn: http://linkedin.com/in/rokeshkannan
- Databricks: https://dbc-463bce5b-f2f7.cloud.databricks.com/dashboardsv3/01f176fce1aa1924bc2a14cde5bcfe9c/published?o=7474655305817874
