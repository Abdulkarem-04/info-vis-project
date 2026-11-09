### 📋 PROJECT DECISION & METHODOLOGY LOG
> **Google Drive (Important):** https://drive.google.com/drive/folders/13VUBJYAd7g0o10NlNFf3rcuP7NGnm5bf?usp=sharing

> **Log Entry #01 — Cumulative Summary (Phases 1 & 2.1)**
> **Timestamp:** Monday, Nov 9, 2026 — 16:05 MYT (08:05 UTC)
> 
> *   **Domain & Dataset Selection (Rubric 3a):** Selected the **Healthcare** domain. Rejected basic beginner datasets in favor of the **Our World in Data (OWID) COVID-19 dataset** (Version 432; 350,085 records; 67 variables). *Justification:* Contains time-series, categorical, and continuous numerical variables required to build all 6 rubric chart types, while exhibiting real-world data messiness necessary for rigorous pre-processing.
> *   **Environment Architecture:** Configured an isolated Python 3.14 virtual environment (`.venv`) on Arch Linux using Jupyter Notebooks in VS Code to ensure reproducibility and comply with PEP 668.
> *   **Format Standardization (Rubric 3b):** Converted the `date` variable from string (`str`) to `datetime64` object to enable chronological sorting and time-series plotting.
> *   **Data-Driven Scoping via Sparsity Audit (Rubric 3a/3b):** Avoided arbitrary country selection and prevented visual "spaghetti charts" (50+ overlapping lines) by conducting a programmatic **Data Sparsity Audit** across all Asian nations on four pillars: Infection (`total_cases`), Mortality (`total_deaths`), Prevention (`total_vaccinations`), and Healthcare Load (`icu_patients`).
> *   **Finding & Country Selection:** Discovered that >90% of Asian countries had `100.0%` missing data for `icu_patients`. Selected **Malaysia (5.9% missing), South Korea (23.0%), Israel (29.3%), and Japan (87.0%)** as the only four nations capable of supporting a complete 4-pillar healthcare capacity analysis. Filtered dataset down to 5,543 focused records (0 duplicate rows found).
> *   **Precision Imputation Strategy (Rubric 3b):** Differentiated between variable types rather than applying a blanket imputation:
>     1. *Monotonic Cumulative Variables* (`total_cases`, `total_deaths`, `total_vaccinations`): Applied country-grouped **Forward Fill (`ffill`)**, as cumulative historical totals cannot decrease during reporting gaps.
>     2. *Point-in-Time Snapshot Variables* (`icu_patients`, `hosp_patients`, `new_cases`, `new_deaths`): Applied country-grouped **Linear Interpolation (`limit=14`)** to estimate smooth, realistic trends across weekly reporting gaps (specifically resolving Japan's once-a-week ICU reporting structure) while preventing cross-country data bleeding and infinite trailing flat-lines. Remaining pre-pandemic leading nulls were filled with `0`.

> **Log Entry #02 — Invalid Observation Audit & Initial Descriptive Stats (Phase 2.2 & 3.1)**
> **Timestamp:** Monday, Nov 9, 2026 — 19:56 MYT (11:56 UTC)
> 
> *   **Invalid & Incorrect Value Audit (Rubric 3b):** Programmatically audited all 5,543 records across the 7 core metrics for biologically impossible negative values (`< 0`), which frequently occur in epidemiological datasets due to retrospective government auditing. Confirmed **0 negative values** across all selected nations (`min = 0.00` for all variables), validating baseline data integrity.
> *   **Initial Distribution Discovery (Rubric 3c):** Descriptive statistics revealed severe **positive (right) skewness** across all active metrics. Most notably, in `new_cases`, the Mean (`14,134.4`) is more than **11x higher** than the Median/`50%` (`1,252.0`), with an extreme maximum spike of `621,328.0` cases in a single day. Similarly, `icu_patients` showed a Median of `121.0` against a Maximum of `3,034.0`. This indicates the presence of extreme statistical outliers that require investigation before visualization.
