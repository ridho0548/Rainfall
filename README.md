# Hydrology Analysis

**© 2025 Ridho Nanda Pratama — Geoscience • Environment • Water • Sustainability**  
**License:** Non-commercial redistribution with attribution; commercial use requires written permission.

## Overview
This repository provides an end-to-end hydrological analytics pipeline to:
- Aggregate and harmonize daily rainfall datasets (primary source: **BMKG**, Indonesia’s Meteorological, Climatological, and Geophysical Agency).
- Perform data quality assurance/quality control (QA/QC), missing value imputation, and outlier detection.
- Conduct multi-year statistical analysis (trends, anomalies, extremes).
- Derive hydrological parameters: **IDF curves**, **ETo** (FAO-56 Penman–Monteith), runoff (SCS-CN), baseflow separation, design discharges (Q2–Q100).
- Generate reproducible visualizations and structured reports.

> *This repository aggregates and harmonizes daily rainfall datasets and provides a reproducible workflow for hydrological statistics, design storms, runoff modeling, and decision-ready reporting.*

---

## Key Features
- **Data Ingestion:** Converts CSV/XLSX rainfall files into a standardized format (ISO date, mm).
- **QA/QC:** Ensures unit consistency, duplicate detection, flatline checks, and physical range validation.
- **Rainfall Statistics:** Daily, monthly, and annual aggregation; percentiles; simple SPI indices.
- **Frequency & Extremes:** Gumbel / Log-Pearson III fitting for design storms and discharges.
- **IDF Curves:** Estimation of rainfall intensity for durations 5–1440 minutes.
- **ETo & Water Balance:** FAO-56 Penman–Monteith, with fallback to Hargreaves when data-limited.
- **Runoff Modeling:** SCS-Curve Number, sensitivity tests, design hydrograph generation.
- **Reporting:** Publication-ready figures (PNG/SVG) and summary tables (Excel/CSV/PDF).
- **Reproducibility:** Environment lock, seeded stochastic methods, and YAML-based configuration.

---

## Directory Structure
```
hydrology-analysis/
├─ data/
│  ├─ raw/                # Raw rainfall and station data
│  ├─ interim/            # Cleaned/standardized intermediates
│  └─ processed/          # Analysis-ready datasets
├─ notebooks/
│  ├─ 01_ingest_qaqc.ipynb
│  ├─ 02_statistics_idf.ipynb
│  └─ 03_runoff_reporting.ipynb
├─ reports/
│  ├─ figures/            # Plots and charts
│  └─ tables/             # Summary tables
├─ src/
│  ├─ config/             # YAML configs
│  ├─ io.py               # Data loaders & writers
│  ├─ qaqc.py             # Validation & cleaning
│  ├─ stats.py            # Statistics & frequency analysis
│  ├─ idf.py              # IDF curve fitting
│  ├─ eto.py              # Reference evapotranspiration
│  ├─ runoff.py           # SCS-CN runoff & peak flow
│  └─ report.py           # Exporting reports
├─ environment.yml        # Conda environment specification
├─ pyproject.toml         # Poetry/pip project dependencies
└─ README.md
```

---

## Quick Installation
Using **conda** (recommended):
```bash
conda env create -f environment.yml
conda activate hydro
```

Using **pip**:
```bash
python -m venv .venv
source .venv/bin/activate  # (Windows: .venv\Scripts\activate)
pip install -U pip
pip install -e .
```

---

## Data Sources
- **BMKG:** Daily rainfall records (mm/day), station metadata (ID, lat, lon, elevation).  
- Optional: TRMM/IMERG (satellite rainfall), local gauge data, streamflow posts.  

> Ensure compliance with source licensing and attribution requirements (e.g., BMKG).

---

## Standard Data Schema
**Rainfall (CSV):**
```text
date,station_id,station_name,lat,lon,elev_m,rain_mm
2018-01-01,960001,Station_A,-6.20,106.82,35,12.4
```

**Station Metadata (CSV):**
```text
station_id,station_name,lat,lon,elev_m,provider
960001,Station_A,-6.20,106.82,35,BMKG
```

---

## Configuration (YAML)
`src/config/project.yaml`:
```yaml
project:
  name: Hydrology Analysis
  period: {start: 2015-01-01, end: 2024-12-31}
data:
  source_dir: data/raw
  out_dir: data/processed
qaqc:
  unit: mm
  missing_codes: [8888, 9999, -99]
  max_daily_mm: 500
frequency:
  dist: log_pearson_iii   # options: gumbel, log_pearson_iii
  return_periods: [2,5,10,25,50,100]
idf:
  durations_min: [5,10,15,30,60,120,180,360,720,1440]
eto:
  method: fao56_pm
runoff:
  method: scs_cn
  cn_default: 75
report:
  figures_dir: reports/figures
  tables_dir: reports/tables
  dpi: 300
```

---

## Workflow
1. **Ingest & QA/QC**  
   ```bash
   python -m src.io --ingest && python -m src.qaqc --fix
   ```
2. **Statistics & IDF**  
   ```bash
   python -m src.stats --annual --extremes && python -m src.idf --fit
   ```
3. **ETo & Runoff**  
   ```bash
   python -m src.eto --compute && python -m src.runoff --design
   ```
4. **Reporting**  
   ```bash
   python -m src.report --export-all
   ```

Or execute the notebooks in the `notebooks/` directory.

---

## Example Usage (Python)
```python
from src.io import load_rain
from src.qaqc import clean_daily
from src.stats import annual_summary
from src.idf import fit_idf, intensity
from src.runoff import scs_qpeak

df = load_rain("data/processed/rainfall.parquet")
dfc = clean_daily(df)
ann = annual_summary(dfc)
idf_model = fit_idf(dfc, durations_min=[5,10,15,30,60])
i_60 = intensity(idf_model, duration_min=60, rp=25)  # mm/h
q_peak = scs_qpeak(area_ha=150, cn=75, i=i_60)
print(i_60, q_peak)
```

---

## Methodology
- **QA/QC:** Range checks, flatline detection, outlier detection (IQR/Z-score), conservative gap filling.  
- **Frequency Analysis:** Gumbel / Log-Pearson III; model selection via AIC/BIC and goodness-of-fit.  
- **IDF Curves:** Duration reduction, regional growth factors (if available), bias checks.  
- **ETo:** FAO-56 Penman–Monteith (with inputs T, RH, u2, Rs/Rn); fallback to Hargreaves.  
- **Runoff (SCS-CN):** Scenario testing for CN values, initial abstraction, design hydrograph estimation.  
- **Uncertainty:** Bootstrap confidence intervals for extremes and IDF fits (optional).  

---

## Outputs
- **Figures:** Annual/multi-year trends, IDF curves, QQ plots, return level plots, hyetographs, hydrographs.  
- **Tables:** Annual summaries, distribution parameters, IDF (duration × return period), design discharges.  
- **Reports:** Lightweight HTML/PDF executive summaries including methodology and limitations.  

---

## Limitations & Ethics
- BMKG and other datasets are subject to licensing; always check redistribution rights.  
- Models carry uncertainty; results are not a substitute for field verification.  
- Users are responsible for safe and responsible application in environmental and infrastructure contexts.  

---

## Contribution Guidelines
1. Fork the repository → create a feature branch → submit a PR with a clear description.  
2. Include example datasets and unit tests for new features.  
3. Follow coding standards (PEP8) and use type hints.  

---

## Citation
If you use this repository in your work, please cite as:  
> Ridho Nanda Pratama (2025). *Hydrology Analysis: Reproducible Pipeline for Rainfall & Runoff Modeling*.  
> URL: (insert repository link)

---

## Contact
- **Author:** Ridho Nanda Pratama  
- **Focus Areas:** Geoscience, Environmental Hydrology, Water Sustainability  
- **Collaboration/Commercial Inquiries:** via LinkedIn or Email

---

## Example CLI Commands
```bash
# Validate and summarize rainfall data (2015–2024)
python -m src.stats --annual --start 2015-01-01 --end 2024-12-31

# Fit IDF curves and export results
python -m src.idf --fit --export

# Compute design discharges for multiple CN values
python -m src.runoff --design --area-ha 143.327 --cn 75 85 95 --rp 2 5 10 25 50 100
```
