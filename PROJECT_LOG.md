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

> **Log Entry #03 — Outlier Detection (IQR) & Temporal Artifact Discovery (Phase 2.3)**
> **Timestamp:** Monday, Nov 9, 2026 — 21:34 MYT (13:34 UTC)
> 
> *   **Mathematical Outlier Detection (Rubric 3b):** Applied the Interquartile Range rule ($\text{Upper Bound} = Q_3 + 1.5 \times \text{IQR}$) across all 7 core metrics. Identified significant statistical outliers across active variables, including `new_cases` (894 rows) and `icu_patients` (627 rows).
> *   **Contextual Outlier Audit:** Inspected the top 50 extreme `new_cases` records and distinguished between two fundamentally different types of outliers:
>     1. *Legitimate Biological Surges:* March 2022 peaks in South Korea (e.g., `621,328` cases on `2022-03-16` with `1,244` ICU patients) and August 2022 peaks in Japan represent true Omicron wave surges (delayed in East Asia due to border controls). *Decision:* Retain these records; deleting them would erase critical healthcare stress events.
>     2. *Reporting Artifacts & Post-Emergency Decay:* Discovered severe anomalies in mid-2023 for South Korea (e.g., `2023-08-01`, `2023-08-14`, `2023-08-21`, `2023-08-28`), where `new_cases` spiked above `250,000–330,000` at 7-day intervals while `icu_patients` read `0.000000`. Identified this as a dual artifact: (a) transition from daily to **weekly batch reporting** (7 days of cases dumped onto a single day), and (b) cessation of ICU reporting after the WHO ended the Public Health Emergency of International Concern (May 2023). Additionally noted fractional human counts (e.g., `1632.714286`) resulting from linear interpolation that require integer rounding.

> **Log Entry #04 — Active Surveillance Window Alignment & Measurement Levels (Phase 1 & 2 Finalization)**
> **Timestamp:** Monday, Nov 9, 2026 — 22:51 MYT (14:51 UTC)
> 
> *   **Temporal Window Audit (Rubric 3b - Invalid Observations):** Audited raw non-null `icu_patients` timestamps across the 4 selected nations:
>     *   *Israel:* `2020-09-01` to `2023-05-07` (979 daily records)
>     *   *Japan:* `2020-05-07` to `2023-10-18` (180 weekly records)
>     *   *Malaysia:* `2020-03-24` to `2023-10-21` (1,306 daily records)
>     *   *South Korea:* `2020-03-28` to `2023-03-04` (1,066 daily records)
> *   **Temporal Cropping Justification:** Identified that South Korea ceased ICU reporting on `2023-03-04` and Israel initiated ICU reporting on `2020-09-01`. Retaining records outside the common intersection (`2020-09-01` to `2023-03-04`) introduces artificial zero-inflation (`icu_patients = 0.0` during South Korea's August 2023 wave), which would severely distort bivariate correlation and scatter plot analyses in Phase 3. Cropped dataset to the strict **915-day common active surveillance window (`2020-09-01` to `2023-03-04`)**.
> *   **Discrete Integer Rounding (Rubric 3b - Incorrect/Inconsistent Values):** Applied `.round(0)` to linearly interpolated patient/case counts to eliminate biologically impossible fractional humans (e.g., `1632.714286` patients in Japan).
> *   **Measurement Levels Classification (Rubric 3a):** Documented statistical measurement scales for the finalized feature set:
>     *   *Nominal (Categorical):* `location`, `iso_code` (qualitative country labels without inherent order).
>     *   *Interval (Temporal):* `date` (chronological timestamps with meaningful intervals but no true biological zero).
>     *   *Ratio (Continuous Numerical):* `total_cases`, `new_cases`, `total_deaths`, `new_deaths`, `total_vaccinations`, `icu_patients`, `hosp_patients`, `population` (quantitative metrics with a true absolute zero allowing multiplicative comparisons).