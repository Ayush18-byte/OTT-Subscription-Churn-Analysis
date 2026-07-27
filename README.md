# 📺 OTT Subscription Churn & Revenue Impact Analysis

### End-to-end churn analytics pipeline turning raw subscription data into revenue-linked retention insights

## 📝 Project Description

This project analyzes subscriber-level data from an OTT (streaming) platform to uncover **why customers churn** and **how much revenue is at stake**. Using SQL extraction, Python-based cleaning, and engineered KPIs, it identifies the specific contract types and plans driving cancellations — and translates those patterns into a targeted, revenue-linked retention strategy.

---

## 🎯 Business Problem

Subscription businesses live and die by retention — acquiring a new customer costs far more than keeping one. The OTT platform behind this dataset had no unified view of **who was churning, why, and what it was costing them**. Customer profiles, subscription/billing details, and support interactions lived in three separate tables with no consolidated reporting layer, making it impossible to prioritize retention efforts or quantify their financial impact.

This project answers three core business questions:

- 📉 **What is the actual churn rate, and which segments churn the most?**
- 💸 **How much recurring revenue and lifetime value is at risk from churn?**
- 🎯 **Which customers should be prioritized for retention outreach, and how?**

---

## 🎯 Project Objectives

- 🔗 Consolidate customer, subscription, and support-ticket data from a normalized SQLite database into a single analysis-ready dataset
- 🧹 Clean and standardize inconsistent fields (missing values, label variants, duplicate records)
- 🛠️ Engineer churn-related features and business KPIs from raw transactional fields
- 📊 Visualize churn trends over time, by plan/contract type, and via correlation analysis
- 💡 Translate analytical findings into a concrete, revenue-linked retention recommendation

---

## 🧰 Tech Stack

| Category | Tools |
|---|---|
| **Language** | Python 🐍 |
| **Data Manipulation** | Pandas, NumPy |
| **Database** | SQLite |
| **Visualization** | Matplotlib, Seaborn |
| **Environment** | Jupyter Notebook |

---

## 🗃️ Dataset Information

| Detail | Description |
|---|---|
| **Raw dataset** | `customer_churn_data_raw.xlsx` — unprocessed export of customer, subscription & support data |
| **Cleaned dataset** | `cleaned_churn_dataset.xls` — final analysis-ready dataset after cleaning & feature engineering |
| **SQLite database** | `customer_churn.db` — normalized relational source of truth |
| **Number of tables** | 3 → `db_customer`, `db_subscription`, `db_support` |
| **Number of customers** | 21 unique customers (post-merge & de-duplication) |
| **Key fields** | `customerid`, `plan_type`, `contract_type`, `monthly_charges`, `cltv`, `churn_score`, `cancellation_date`, `escalations` |

---

## 📁 Repository Structure

```
OTT-Subscription-Churn-Analysis/
│
├── OTT_Subscription_Churn_Analysis.ipynb   # Main analysis notebook (end-to-end pipeline)
├── customer_churn.db                        # SQLite source database (3 tables)
├── customer_churn_data_raw.xlsx              # Raw exported dataset
├── cleaned_churn_dataset.xls                 # Cleaned, feature-engineered dataset
├── requirements.txt                          # Python dependencies
└── README.md                                 # Project documentation (this file)
```

---

## 🔄 Project Workflow

```
Raw Data
   ↓
SQLite Database
   ↓
Data Extraction
   ↓
Data Cleaning
   ↓
Feature Engineering
   ↓
KPI Calculation
   ↓
Data Visualization
   ↓
Business Insights
```

Each stage is implemented as its own section inside `OTT_Subscription_Churn_Analysis.ipynb`, so the notebook can be read top-to-bottom as a fully reproducible pipeline.

---

## 🧹 Data Cleaning

The raw tables required meaningful cleanup before they were analysis-ready:

- **Schema inspection** — programmatically read table names and column metadata directly from `sqlite_master` and `PRAGMA table_info`
- **Column pruning** — dropped sparse/unusable columns (`interests`, `pincode`) with minimal non-null coverage
- **Renaming** — renamed ambiguous columns (e.g. `name` → `customer_name`) for clarity
- **Type conversion** — converted `dob`, `subscription_start_date`, `cancellation_date`, and `renewal_date` to proper `datetime` types
- **Label standardization** — normalized inconsistent gender labels (`Men`/`Women` → `Male`/`Female`)
- **Missing value imputation** — filled missing `country` values using a state-to-country lookup built from complete records
- **De-duplication** — collapsed duplicate support tickets to one row per customer using a `complaint_count` derived via `groupby`, retaining the most representative record
- **Table merging** — joined `db_subscription`, `db_customer`, and `db_support` on `customerid` (left joins) into a single unified DataFrame

---

## 🛠️ Feature Engineering

New fields were engineered to make churn measurable and actionable:

| Feature | Description |
|---|---|
| `churn_flag` | Binary flag (1/0) derived from whether `cancellation_date` is populated |
| `tenure_days` | Days between subscription start and cancellation (or today, for active users) |
| `churn_risk` | Rule-based tier (`low` / `med` / `high`) built from `churn_score` thresholds via `np.select` |
| `complaint_count` | Number of support tickets per customer, derived via `groupby` on `db_support` |
| `revenue_at_risk` | Sum of `monthly_charges` for all customers flagged as churned |

---

## 📊 Business KPIs

| KPI | Value |
|---|---|
| Overall Churn Rate | **28.6%** |
| Retention Rate | **71.4%** |
| Monthly-Contract Churn Rate | **55.6%** |
| Annual-Contract Churn Rate | **8.3%** |
| Basic Plan Churn Rate | **60.0%** |
| Standard Plan Churn Rate | **22.2%** |
| Premium Plan Churn Rate | **14.3%** |
| ARPU (Average Revenue Per User) | **$18.85** |
| Average Customer Tenure | **~1,504 days** |
| Revenue at Risk (MRR) | **$73.94 / month** |
| CLTV Erosion (6 at-risk customers) | **$2,047** |
| Escalation Rate | **19.05%** |
| Avg. Complaints per Customer | **0.43** |

---

## 🔍 Exploratory Data Analysis

The EDA phase focused on isolating *where* churn concentrates and *what* correlates with it:

- 📅 **Time-based analysis** — cancellations were grouped by month to trace churn volume trends and flag anomalous spikes
- 📦 **Segment analysis** — churn rate was broken down by `plan_type` and `contract_type` to identify the highest-risk subscriber segments
- 🔗 **Correlation analysis** — a correlation matrix across `plan_type`, `contract_type`, `churn_score`, `churn_flag`, and `churn_risk` was built (after encoding categorical fields) to validate which factors most strongly predict churn
- 🚦 **Risk segmentation** — customers were distributed across `low` / `med` / `high` churn-risk tiers to support prioritized outreach

---

---

## 💡 Key Findings

- 📉 The platform's overall churn rate stands at **28.6%**, with a corresponding retention rate of **71.4%**
- 📆 **Monthly-contract subscribers churn at 55.6%** — nearly **6.7×** the **8.3%** churn rate seen among annual-contract subscribers
- 🥉 **Basic-plan subscribers churn at 60.0%**, more than **4×** the Premium plan's **14.3%** rate
- 🔗 The correlation analysis shows `churn_score` and `churn_risk` are very strongly associated with actual churn outcomes, while `contract_type` shows a meaningful negative correlation with churn — reinforcing that annual contracts are protective
- 💰 **$73.94/month** in MRR and **$2,047** in CLTV are currently at risk from just **six** identified customers
- 📞 The escalation rate sits at **19.05%**, with an average of **0.43 complaints per customer**, indicating support friction is a contributing — though not dominant — churn factor

---

## 🚀 Business Recommendations

Because monthly-contract subscribers churn at nearly **7×** the rate of annual-contract subscribers, and Basic-plan users churn at over **4×** the rate of Premium users, the highest-leverage retention lever is a **targeted contract-migration campaign**:

- 🎯 Proactively offer discounted annual-contract upgrades to **high churn-risk**, monthly-billed Basic/Standard customers — identified via the `churn_risk` tier — *before* they reach their renewal date
- 💵 Prioritize outreach to the customers currently contributing to the **$73.94/month MRR** and **$2,047 CLTV** at risk, since converting even a fraction of this at-risk monthly base to annual contracts would materially reduce revenue leakage
- 🛎️ Pair contract-migration offers with proactive support check-ins for customers with elevated `complaint_count` or `escalations`, given their correlation with dissatisfaction-driven churn

---

## ▶️ How to Run the Project

1. Ensure `customer_churn.db` is in the project root (already included in this repo)
2. Open `OTT_Subscription_Churn_Analysis.ipynb` in Jupyter
3. Run all cells sequentially — the notebook will:
   - Connect to `customer_churn.db` and load all three tables
   - Clean, merge, and feature-engineer the dataset
   - Calculate all business KPIs
   - Generate all visualizations shown above
4. Optionally, export the final cleaned dataset (already provided as `cleaned_churn_dataset.xls`) for downstream reporting or dashboarding tools

---

## 🔮 Future Improvements

- 🤖 Build a predictive churn model (e.g. logistic regression / gradient boosting) using `churn_score` and engineered features as inputs
- 📊 Connect the cleaned dataset to a live BI dashboard (Power BI / Tableau) for stakeholder self-service
- 🗂️ Scale the pipeline to handle larger, production-sized subscriber datasets
- 🔁 Automate the SQLite → cleaned-dataset pipeline as a scheduled ETL script
- 🧪 A/B test the proposed contract-migration offer against a control group to measure real-world retention lift

---

## 🧠 Skills Demonstrated

- 🧹 Data Cleaning
- 🗃️ Data Wrangling
- 🛠️ Feature Engineering
- 🗄️ SQL
- 🔍 Exploratory Data Analysis (EDA)
- 📊 Business Analytics
- 📈 Data Visualization
- 📌 KPI Development

---

## ⭐ Repository Highlights


- ✅ Works directly from a **relational SQLite database**, not a pre-cleaned CSV — reflecting real-world messiness
- ✅ Every KPI is **traceable back to code**, with no black-box numbers
- ✅ Findings are translated into a **specific, revenue-quantified business recommendation** — not just charts
- ✅ Clean, modular notebook structure that mirrors a real analytics workflow: extract → clean → engineer → analyze → visualize → recommend
- ✅ Recruiter-friendly documentation that connects technical work directly to business outcomes

----