# Climate Extremes, Inflation and Real Wages in Argentina (SQL + DuckDB)

This repository contains an end-to-end analytics project that integrates **official Argentine economic indicators** with a **monthly climate proxy** to study how **climate extremes** relate to:
- consumer price inflation (overall vs. food), and
- real wages (RIPTE deflated by IPC).

The core of the project is a **DuckDB** analytical model built from raw CSV inputs, queried with **SQL** (window functions, dimensional modeling, benchmarking vs. national averages).

---

## Project questions

1. **Same-month effect:** Do months classified as climate extremes show different inflation / real wage outcomes?
2. **Lagged effect (1 month):** Does the economic response occur one month after an extreme?
3. **Regional heterogeneity:** Do regions deviate from the national benchmark during climate extremes (food inflation)?

---

## Data sources

### 1) INDEC — IPC (Índice de Precios al Consumidor)
- File used: `serie_ipc_aperturas.csv`
- Granularity: monthly, by **COICOP category** (“aperturas”) and by **region**

Saved to:
- `data/raw/serie_ipc_aperturas.csv`

### 2) RIPTE — Remuneración Imponible Promedio de los Trabajadores Estables
- File used: `remuneracion-imponible-promedio-trabajadores-estables-ripte-total-pais-pesos-serie-mensual.csv`
- Granularity: monthly, total country (nominal ARS)

Saved to:
- `data/raw/ripte.csv` (renamed for convenience)

### 3) Meteostat (climate proxy)
- Data pulled via the `meteostat` Python package
- Method: daily temperatures for selected major cities → daily mean across cities → monthly aggregation
- Output: `data/processed/climate_monthly_avg.csv`

---

## Repository structure

```
.
├─ notebooks/
│  ├─ 02_climate_processing_annotated.ipynb
│  └─ 03_sql_data_model_annotated.ipynb
├─ data/
│  ├─ raw/
│  │  ├─ serie_ipc_aperturas.csv
│  │  └─ ripte.csv
│  └─ processed/
│     ├─ climate_monthly_avg.csv
│     └─ analytics.duckdb
└─ README.md
```

---

## How to run

### Step 1 — Create climate monthly dataset
Open and run:
- `notebooks/02_climate_processing_annotated.ipynb`

This generates:
- `data/processed/climate_monthly_avg.csv`

### Step 2 — Build DuckDB model and run analyses
Open and run:
- `notebooks/03_sql_data_model_annotated.ipynb`

This creates / updates:
- `data/processed/analytics.duckdb`

---

## Key SQL techniques demonstrated

- **Staging → cleaning → typed tables** (ETL in SQL)
- **Dimensional modeling**
  - `dim_month`
  - fact tables at multiple grains (national and regional)
- **Analytical SQL**
  - window functions (`LAG`, `FIRST_VALUE`)
  - benchmarking vs. a national baseline
  - regime-based aggregation

---

## Main results (high level)

### A) Same-month relationship (climate regime vs outcomes)
- Heat extremes coincide with **higher food inflation** and **lower real wages** compared to normal months.

### A-extended) Lagged relationship (1-month lag)
- Months following a heat extreme show a **stronger** signal, especially in **food inflation**.

### B) Regional heterogeneity (food inflation vs national benchmark)
- Under heat extremes, the deviation from the national average is heterogeneous across regions
  (some regions above national benchmark, others below), while normal months show convergence.

> Note: extreme regimes are rare by construction (z-score threshold), so the sample size for extremes can be small.
> The notebook includes a clear regime definition and reproducible queries so thresholds can be adjusted and re-tested.

---

## Notes / assumptions

- Climate is a **national proxy** built by averaging multiple city series; it is intended for a first-pass national signal.
- “Real wage” is implemented as:  
  `real_wage_index = (RIPTE / IPC_NivelGeneral) * 100`, then re-based to 100 at the first available month.

---

## Reproducibility

- All transformations are performed in notebooks.
- DuckDB is used as an embedded analytics database to keep the project lightweight and easy to run locally.

