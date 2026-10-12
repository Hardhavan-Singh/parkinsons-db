# Parkinson's Progression Biomarker Database

A normalized relational database for longitudinal clinical and
cerebrospinal-fluid proteomic data from Parkinson's disease patients,
with a Streamlit query interface.

---

## Problem Statement

The AMP-PD study data is distributed as four denormalized CSV files with
no integrity enforcement, no relational structure, and no query
interface. This creates four concrete problems:

1. **Repeating groups** — MDS-UPDRS Parts 1–4 are stored as four separate
   columns, violating 1NF. Adding a fifth part would require a schema
   change.
2. **Redundancy** — `patient_id`, `visit_month` and `UniProt` are
   repeated across ~1.2 million rows, creating update anomalies.
3. **No integrity enforcement** — flat files cannot prevent orphaned
   measurements, out-of-range clinical scores, or duplicate readings.
4. **No query capability** — answering a research question requires
   writing an ad-hoc script every time.

This project converts that data into a normalized relational schema with
enforced constraints, indexed access paths, role-based access control,
audit logging, and a declarative query interface.

---

## Dataset

**Source:** AMP®-Parkinson's Disease Progression Prediction (Kaggle)  
**Origin:** Accelerating Medicines Partnership® Parkinson's Disease,
managed through the Foundation for the National Institutes of Health.

| File | Rows | Size |
|---|---|---|
| `train_peptides.csv` | 981,834 | 51.4 MB |
| `train_proteins.csv` | 232,741 | 7.7 MB |
| `train_clinical_data.csv` | 2,615 | 74 KB |
| `supplemental_clinical_data.csv` | 2,223 | 76 KB |

**Profiled characteristics:**

- 248 patients with CSF samples across 1,113 sampled visits
- 227 distinct proteins, 968 distinct peptides
- Each peptide maps to exactly one protein — a clean 1:M relationship
- `(visit_id, UniProt)` and `(visit_id, UniProt, Peptide)` are unique,
  validating composite primary keys
- Zero nulls in both measurement files
- `visit_id` is a derived key: `patient_id` + `_` + `visit_month`

**Known data-quality anomalies (handled explicitly in ETL):**

- 45 visits appear in the proteomic files with no matching clinical
  record — a referential integrity violation in the source data
- The 771 supplemental patients are entirely disjoint from the 248
  clinical patients, with no shared identifiers
- `updrs_4` is null in 40% of records and medication state in 51%;
  these are semantically "not assessed", not missing values

Raw data is **not committed** to this repository. See
`docs/data_setup.md` for acquisition instructions.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Database | PostgreSQL 18.6 |
| ETL | Python 3.14, pandas, psycopg 3 |
| Interface | Streamlit, Plotly |
| Tooling | pgAdmin 4, VS Code, Git |

---

## Repository Structure
parkinsons-db/
├── data/raw/ Source CSVs (gitignored)
├── sql/
│ ├── 01_schema.sql DDL — tables and primary keys
│ ├── 02_constraints.sql Foreign keys, CHECK constraints, triggers
│ ├── 03_indexes.sql Index definitions
│ └── 04_queries.sql Documented query set
├── etl/ Profiling and load scripts
├── app/ Streamlit application
├── docs/
│ ├── report/ Project documentation
│ └── diagrams/ ER diagram and exports
├── .env.example Connection template (no credentials)
├── requirements.txt
└── README.md


SQL files are numbered to run in dependency order: schema before
constraints before indexes.

---

## Setup

```bash
git clone https://github.com/Hardhavan-Singh/parkinsons-db.git
cd parkinsons-db

python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt

copy .env.example .env
REM edit .env with your local PostgreSQL password

psql -U postgres -c "CREATE DATABASE parkinsons_db;"
psql -U postgres -d parkinsons_db -f sql/01_schema.sql
psql -U postgres -d parkinsons_db -f sql/02_constraints.sql
python etl/load_data.py
```

Database credentials are read from environment variables via `.env` and
are never committed to version control.

---

## Progress

- [x] Environment setup (PostgreSQL 18.6, Python 3.14, Git)
- [x] Dataset acquired and profiled
- [ ] ER model
- [ ] Normalization to 3NF (documented with functional dependencies)
- [ ] Schema DDL and constraints
- [ ] ETL pipeline
- [ ] Query set (15–20 documented queries)
- [ ] Indexing and performance benchmarks
- [ ] Role-based access control
- [ ] Audit logging via triggers
- [ ] Streamlit interface
- [ ] Documentation

---

## Scope and Limitations

This project builds data infrastructure. It makes no clinical or
diagnostic claims.

- Single-node deployment; no replication or distributed querying
- Batch ETL only; no real-time or streaming ingestion
- The Kaggle release omits patient demographics (age, sex, cohort),
  which are available only through the full AMP-PD portal under
  institutional access
- Findings reflect one study cohort and are not generalizable

---

## Acknowledgement

Data courtesy of the Accelerating Medicines Partnership® Parkinson's
Disease initiative. Used for academic purposes only.