# Healthcare Data Pipeline

A hands-on data engineering project built to learn end-to-end pipeline development using industry-standard tools. This project implements a **Medallion Architecture (Bronze / Silver / Gold)** for a synthetic healthcare dataset, with separate DEV and PROD environments hosted on DigitalOcean.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **Apache HOP 2.15.0** | Pipeline development and orchestration |
| **PostgreSQL 15** | Data warehouse (Bronze, Silver, Gold schemas) |
| **Docker / Docker Compose** | Container management on cloud servers |
| **DigitalOcean** | DEV and PROD Ubuntu 24.04 cloud servers |
| **GitHub** | Version control and CI/CD (in progress) |
| **pgAdmin** | Database querying and verification |

---

## Architecture

This project follows the **Medallion Architecture** pattern — data flows through three layers of increasing quality and refinement:

```
Source CSV Files
      │
      ▼
┌─────────────┐
│   BRONZE    │  Raw, unmodified source data
│  (raw_*)    │  All fields stored as TEXT
└─────────────┘
      │
      ▼
┌─────────────┐
│   SILVER    │  Cleansed, validated, typed data
│             │  Duplicates removed, dates cast
└─────────────┘
      │
      ▼
┌─────────────┐
│    GOLD     │  Aggregated, business-ready data
│             │  Materialized Views for performance
└─────────────┘
```

---

## Infrastructure

Two separate DigitalOcean droplets — DEV and PROD are never the same server.

| Environment | Purpose | Database |
|---|---|---|
| **DEV** | Development and testing with synthetic data | `healthcare_dw_dev` |
| **PROD** | Production environment | `healthcare_dw` |

Pipelines are built and tested on DEV first. Changes flow one direction only: **DEV → GitHub → PROD**. Real patient data never touches DEV.

---

## Project Structure

```
healthcare-data-pipeline/
├── .hop/                          # HOP project metadata
│   └── project-config.json
├── pipelines/
│   ├── bronze/                    # Bronze ingestion pipelines (.hpl)
│   │   ├── bronze_load_patient.hpl        ✅ Complete
│   │   ├── bronze_load_encounters.hpl     🔲 In progress
│   │   ├── bronze_load_diagnoses.hpl      🔲 In progress
│   │   └── bronze_load_lab_results.hpl    🔲 In progress
│   ├── silver/                    # Silver cleansing pipelines
│   └── gold/                      # Gold aggregation pipelines
├── workflows/                     # HOP orchestration workflows (.hwf)
├── sql/
│   ├── bronze/                    # Bronze table DDL
│   ├── silver/                    # Silver table DDL
│   └── gold/                      # Gold table DDL + Materialized Views
├── data/
│   └── samples/                   # Synthetic CSV source files
│       ├── patients.csv
│       ├── encounters.csv
│       ├── diagnoses.csv
│       └── lab_results.csv
├── docs/                          # Architecture and data dictionary
├── .gitignore
└── README.md
```

---

## Pipeline Status

### Bronze Layer — Raw Ingestion

Each Bronze pipeline follows the same four-step pattern:
`CSV File Input → Filter Rows → Add Constants → Table Output`

| Pipeline | Target Table | Status |
|---|---|---|
| `bronze_load_patient` | `bronze.raw_patient` | ✅ Complete — 5 rows loaded, 1 bad row filtered |
| `bronze_load_encounters` | `bronze.raw_encounter` | 🔲 In progress |
| `bronze_load_diagnoses` | `bronze.raw_diagnosis` | 🔲 In progress |
| `bronze_load_lab_results` | `bronze.raw_lab_result` | 🔲 In progress |

### Silver Layer — Cleansing & Validation
🔲 Not started — begins after all Bronze pipelines are complete

### Gold Layer — Aggregation & Materialized Views
🔲 Not started

---

## Sample Data

Synthetic data simulating daily EHR extracts. Located in `data/samples/`.

| File | Records | Notes |
|---|---|---|
| `patients.csv` | 6 rows | Includes 1 intentionally bad row (missing MRN) for filter testing |
| `encounters.csv` | 4 rows | Includes a readmission scenario |
| `diagnoses.csv` | 5 rows | ICD-10 coded |
| `lab_results.csv` | 4 rows | LOINC coded with abnormal flags |

---

## Environment Configuration

HOP environments are defined locally and **never committed to GitHub**. Two environment config files are required to run pipelines:

- `DEV-config.json` — points to DEV droplet
- `PROD-config.json` — points to PROD droplet

Both files are excluded via `.gitignore`. Pipelines reference environment variables (`${HOP_DB_HOST}`, `${HOP_DB_NAME}`, etc.) so the same pipeline file runs against either environment without modification.

---

## Security Notes

- Credentials are stored in HOP environment config files only — never hardcoded in pipeline files
- Config files containing passwords are excluded from version control via `.gitignore`
- PostgreSQL port 5432 is restricted by UFW firewall — not open to the public internet
- DEV and PROD use separate database users (`hop_dev` / `hop_prod`) with least-privilege access

---

## Roadmap

- [x] Infrastructure setup — DEV and PROD droplets, Docker, PostgreSQL, HOP
- [x] Medallion schemas created (Bronze / Silver / Gold) on both environments
- [x] First Bronze pipeline — patient ingestion
- [ ] Remaining Bronze pipelines (encounters, diagnoses, lab results)
- [ ] Silver layer pipelines with validation and upsert logic
- [ ] Gold layer aggregations and Materialized Views
- [ ] Master orchestration workflow (`main_daily_pipeline.hwf`)
- [ ] GitHub Actions CI/CD — automated deployment to DEV on push
- [ ] Scheduled pipeline execution on server
- [ ] AI enablement — PDF ingestion from Azure Data Lake (planned)

---

## Key Lessons Learned

- HOP environment config files (`DEV-config.json`, `PROD-config.json`) must be in `.gitignore` — the course material omitted this step
- The `Add Constants` step should not include `load_timestamp` when the database column already has `DEFAULT NOW()` — doing so causes a parse error
- DEV and PROD must always be separate servers — a single mistake on a shared server can corrupt irreplaceable data

---

## Learning Resources

- [Apache HOP Documentation](https://hop.apache.org/manual/latest/)
- [Apache HOP YouTube — Build & Deploy Series](https://www.youtube.com/playlist?list=PL1OStsJJPxv4aOWNqcSTygZ-A_LCWujUq)
- [PostgreSQL 15 Documentation](https://www.postgresql.org/docs/15/)
