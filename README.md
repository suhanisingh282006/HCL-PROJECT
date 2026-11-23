# Patient Readmission Analysis (30-Day Healthcare Analytics Pipeline)

[![Python](https://img.shields.io/badge/Python-Pandas%20%7C%20NumPy-0F766E?logo=python)](https://www.python.org/)
[![SQL](https://img.shields.io/badge/SQL-Oracle%20%7C%20MySQL%208.0-D97706?logo=mysql)](https://www.mysql.com/)
[![Power BI](https://img.shields.io/badge/Power%20BI-Desktop%20Dashboard-F2C811?logo=powerbi)](https://powerbi.microsoft.com/)
[![Domain](https://img.shields.io/badge/Domain-Healthcare%20Analytics-115E59)]()

> **HCL Data Analyst Capstone — Project #16**  
> **Author:** Suhani Singh (`2500321540194`)  
> **Repository:** [https://github.com/suhanisingh282006/HCL-PROJECT](https://github.com/suhanisingh282006/HCL-PROJECT)  
> **Dataset:** Diabetes 130-US Hospitals for Years 1999–2008 (Kaggle / UCI)

---

## 1. Problem Statement & Project Scope
Unplanned hospital readmissions within 30 days of discharge serve as a critical benchmark for inpatient care quality and preventable healthcare expenditure. This repository implements an end-to-end clinical analytics pipeline across **Python**, **Relational SQL (Oracle SQL / MySQL 8.0)**, and **Power BI** to:
1. Clean raw inpatient encounter records and resolve edge cases (**missing patient identifiers, expired patient dispositions, inter-facility transfers, and longitudinal multi-visit histories**).
2. Compute and cross-verify **30-day unplanned readmission flags** using both chronological windowing in Pandas and correlated **30-day self-joins** in SQL.
3. Deliver an executive **Power BI dashboard** tracking readmission risk across age brackets and primary diagnosis groups.

---

## 2. Executive KPI Summary (Verified Results)

| KPI Metric | Verified Value | Definition / Clinical Notes |
| :--- | :---: | :--- |
| **Raw Encounters Loaded** | `101,766` | Source baseline (`diabetic_data.csv`) |
| **Expired Dispositions Removed** | `1,652` | Excluded (`discharge_disposition_id IN (11, 19, 20, 21)`) |
| **Clean Eligible Encounters** | `100,114` | Validated denominator exported to `clean_admissions.csv` |
| **Facility Transfers Isolated** | `18,081` (`18.06%`) | Flagged (`is_transfer = 1`, IDs `2, 3, 4, 5`), excluded from unplanned returns |
| **30-Day Unplanned Readmissions** | `4,067` | Non-transfer return visits within `1–30` days of discharge |
| **Overall 30-Day Readmission Rate** | **`4.06%`** | Reconciled identically across Python, SQL, and Power BI |

---

## 3. Data Model (Star Schema)

```mermaid
erDiagram
    FACT_ADMISSIONS {
        BIGINT encounter_id PK
        BIGINT patient_nbr FK
        INT diag_id FK
        DATE admission_date FK
        DATE discharge_date
        INT time_in_hospital
        TINYINT is_transfer
        TINYINT readmit_30d_flag
    }
    DIM_PATIENT {
        BIGINT patient_nbr PK
        VARCHAR age_group
        VARCHAR gender
    }
    DIM_DIAGNOSIS {
        INT diag_id PK
        VARCHAR icd9_code
        VARCHAR diagnosis_group
    }
    DIM_DATE {
        DATE date_key PK
        INT month
        INT quarter
        INT year
    }

    DIM_PATIENT ||--o{ FACT_ADMISSIONS : "has"
    DIM_DIAGNOSIS ||--o{ FACT_ADMISSIONS : "classified_by"
    DIM_DATE ||--o{ FACT_ADMISSIONS : "admitted_on"
