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

> **Log Entry #05 — Phase 2 Completion & Dataset Export (Milestone 1)**
> **Timestamp:** Tuesday, Nov 10, 2026 — 15:36 MYT (07:36 UTC)
> 
> *   **Finalized Clean Dataset (`covid_cleaned.csv`):** Successfully cropped, rounded, and exported the finalized analytical dataset.
>     *   *Dimensions:* **3,660 records** across **10 variables** (`location`, `date`, 7 core epidemiological metrics, and `population`).
>     *   *Class Balance / Frequency (Rubric 3c):* Achieved an exact **1:1:1:1 balanced panel distribution** across the 4 nations—**915 daily observations per country** (`Israel`: 915, `Japan`: 915, `Malaysia`: 915, `South Korea`: 915) spanning `2020-09-01` to `2023-03-04`.
> *   **Version Control & Reproducibility:** Configured `.gitignore` to exclude the 91.89 MB raw CSV and local `.venv` binaries while committing `phase1_analysis.ipynb`, `covid_cleaned.csv`, and `PROJECT_LOG.md` to Git (`commit b356834`).

> **Log Entry #06 — Population Normalization & The Scale Reversal Paradox (Phase 3.1)**
> **Timestamp:** Tuesday, Nov 10, 2026 — 18:18 MYT (10:18 UTC)
> 
> *   **Feature Engineering for Comparative EDA (Rubric 3c):** Engineered four population-normalized metrics (`cases_per_million`, `icu_per_million`, `deaths_per_million`, `vax_per_hundred`) and updated `covid_cleaned.csv` (now 3,660 rows × 14 columns) to eliminate population-size confounding across nations ranging from 9.45M (Israel) to 123.95M (Japan).
> *   **EDA Discovery #1 (The ICU Scale Reversal):** Country-grouped descriptive statistics revealed a complete rank reversal between raw and population-adjusted ICU loads:
>     *   *Raw Mean ICU Load:* Japan appeared highest (`994.94` patients/day) and Israel lowest (`124.59` patients/day).
>     *   *Normalized Mean ICU Load (Per Million):* **Israel actually experienced the highest healthcare strain (`13.19` ICU patients/million)**, followed by Malaysia (`10.21`), while Japan (`8.03`) and South Korea (`6.84`) had the lowest per-capita ICU burden. This proves that raw counts create a false narrative due to Japan's 13x larger population.
> *   **Multi-Dose Vaccination Interpretation:** Observed `Max_Vax_Doses_Per_100` exceeding 100 across all nations (Japan: `308.68`, South Korea: `250.21`, Malaysia: `213.69`, Israel: `197.32`). Because `total_vaccinations` tracks cumulative syringe doses administered rather than unique individuals, values of `200–308` reflect population-wide completion of a **2-dose primary series plus 1st and 2nd booster campaigns** (~2 to 3.1 doses per citizen).

> **Log Entry #07 — Bivariate Correlation, Clinical Decoupling, & Skewness Analysis (Phase 3.2)**
> **Timestamp:** Tuesday, Nov 10, 2026 — 21:56 MYT (13:56 UTC)
> 
> *   **Correlation Analysis (Rubric 3c):** Computed the Pearson correlation matrix ($r$) across population-normalized metrics (`cases_per_million`, `icu_per_million`, `deaths_per_million`, `vax_per_hundred`):
>     1. *The ICU Mortality Bottleneck ($r = 0.789$):* Discovered a strong positive correlation between `icu_per_million` and `deaths_per_million` ($0.789$), significantly higher than the moderate correlation between `cases_per_million` and `deaths_per_million` ($0.475$). This proves statistically that ICU capacity strain—rather than raw infection volume—is the primary clinical predictor of mortality.
>     2. *Clinical Decoupling via Vaccination ($r = +0.304$ vs. $r = -0.122$):* Uncovered a critical epidemiological phenomenon. While `vax_per_hundred` correlated positively with `cases_per_million` ($+0.304$)—because highly transmissible Omicron waves occurred in 2022 after vaccination rollouts—`vax_per_hundred` exhibited a **negative correlation with `icu_per_million` ($-0.122$)** and near-zero correlation with `deaths_per_million` ($0.059$). This demonstrates that vaccination successfully decoupled infection spread from severe ICU hospitalization.
> *   **Distribution Skewness Analysis (Rubric 3c):** Confirmed severe positive skewness ($> 1.0$) across `cases_per_million` (`5.43`), `deaths_per_million` (`3.52`), and `icu_per_million` (`1.74`), driven by acute pandemic wave surges. Conversely, `vax_per_hundred` displayed a near-symmetrical distribution (`-0.26`) reflecting steady, cumulative rollout campaigns.

> **Log Entry #08 — Categorical Feature Engineering & ICU Stress Frequency Analysis (Phase 3.3)**
> **Timestamp:** Tuesday, Nov 10, 2026 — 22:23 MYT (14:23 UTC)
> 
> *   **Ordinal Discretization (`icu_stress_level`):** Applied `pd.cut()` to discretize continuous `icu_per_million` values into four clinical severity regimes: `1_Low (<5/M)`, `2_Moderate (5-15/M)`, `3_High (15-25/M)`, and `4_Critical (>25/M)`. Saved the 15-column dataset to `covid_cleaned.csv`.
> *   **Frequency & Cross-Tabulation Analysis (Rubric 3c):** Across all 3,660 country-days, hospitals operated in Low or Moderate stress for `81.0%` of the timeline (`39.5%` Low, `41.5%` Moderate), while experiencing Critical Overload (`>25/M`) on `305 days (8.3%)`.
> *   **Country-Level Vulnerability Discovery:** Cross-tabulation exposed a stark divergence in healthcare system resilience:
>     *   **Israel (`164 days`)** and **Malaysia (`138 days`)** accounted for **99.0% (302 out of 305 days)** of all Critical ICU Overload events in the dataset.
>     *   Conversely, **Japan (`0 days`)** and **South Korea (`3 days`)** almost entirely avoided critical per-capita ICU exhaustion, successfully containing their hospital load within Low and Moderate regimes (`831/915 days` for Japan; `830/915 days` for South Korea).

> **Log Entry #09 — Visualization #1 (Histogram) & Visual Occlusion Analysis (Phase 3.4 & 4.1)**
> **Timestamp:** Monday, Nov 16, 2026 — 15:40 MYT (07:40 UTC)
> 
> *   **Visualization #1 Selection & Justification (Rubric 3c & 3d - Histogram):** Constructed an interactive multi-country Histogram (`px.histogram`, `nbins=30`, `opacity=0.65`) of `icu_per_million` with a clinical threshold marker at `25/M`. *Selection Justification:* A histogram is the required univariate/multivariate visualization for inspecting continuous frequency distributions, empirical skewness (`1.74`), and tail density past critical healthcare capacity thresholds.
> *   **Information Design Critique (Overplotting / Visual Occlusion):** Identified that overlaying 4 semi-transparent country distributions on a single axis creates chromatic blending in the low-to-moderate range (`0–25/M`), increasing cognitive load when distinguishing overlapping bars without interactive hover/legend filtering. This directly motivates pairing the Histogram with **Visualization #2 (Box Plot)** to achieve non-overlapping, side-by-side quartile comparison.

> **Log Entry #10 — Visualization #2 (Box Plot) & IQR Outlier Mechanics (Phase 3.5 & 4.2)**
> **Timestamp:** Monday, Nov 16, 2026 — 16:13 MYT (08:13 UTC)
> 
> *   **Visualization #2 Selection & Justification (Rubric 3c & 3d - Box Plot):** Developed a categorical Box Plot (`px.box`) of `icu_per_million` across the 4 nations. *Selection Justification:* Eliminates the visual occlusion of overlaid histograms by isolating each country into a non-overlapping vertical lane, enabling direct comparison of medians (`50%`), interquartile spreads ($\text{IQR} = Q_3 - Q_1$), upper fences ($Q_3 + 1.5 \times \text{IQR}$), and extreme outlier days.
> *   **Statistical Discovery (Israel vs. Malaysia Distribution Structure):**
>     *   *Malaysia:* Exhibited the lowest Median (`4.51/M`) and a relatively compact box ($Q_1 = 1.63$, $Q_3 = 12.29$, $\text{IQR} = 10.66$), resulting in an upper fence of `27.43/M`. Consequently, Malaysia's peak crisis days (`27.5` to `46.94/M`) were mathematically classified as a dense cluster of **extreme upper outliers**.
>     *   *Israel:* Exhibited the highest Median (`9.58/M`) and the widest interquartile spread ($Q_3 = 20.00/M$), pushing its upper fence to `~43.1/M`. Because high ICU loads occurred repeatedly across Israel's timeline, values between `25/M` and `43/M` were absorbed into its normal whisker range rather than flagged as outliers.

> **Log Entry #11 — Visualization #3 (Line Chart) & Longitudinal Wave Dynamics (Phase 4.3)**
> **Timestamp:** Monday, Nov 16, 2026 — 20:16 MYT (12:16 UTC)
> 
> *   **Visualization #3 Selection & Justification (Rubric 3d - Line Chart):** Constructed a multi-series longitudinal Line Chart (`px.line`) plotting `icu_per_million` over `date` (`2020-09-01` to `2023-03-04`). *Selection Justification:* Box plots and histograms aggregate away chronological order; a continuous time-series line chart is essential to reveal epidemic wave frequency, duration, seasonal timing, and post-intervention trajectory changes.
> *   **Longitudinal Discovery (Explaining the Box Plot Paradox):**
>     *   *Israel (Recurrent Multi-Wave Pattern):* Breached the `25/M` critical threshold **four separate times** (Oct 2020, Jan–Mar 2021, Sep 2021, and Feb 2022), explaining why its Box Plot interquartile range ($Q_3 = 20.0$) and upper fence (`43.1/M`) were stretched high.
>     *   *Malaysia (Single Concentrated Surge + Post-Vaccine Suppression):* Breached the `25/M` threshold **only once** during a sustained 5-month Delta wave (`May 2021 – Oct 2021`, peaking at `46.94/M`). Following the completion of Malaysia's rapid adult vaccination drive in late 2021, Malaysia's ICU load dropped to the **lowest of all four nations** throughout 2022 and 2023—explaining why Malaysia had the lowest overall Median (`4.51/M`) and why its 2021 Delta surge appeared as isolated upper outliers in Chart 2.
>     *   *(Side Note Recorded):* Scheduled a comprehensive "Study & Assessment Mode" review session upon project completion to solidify theoretical mastery.

> **Log Entry #12 — Visualization #4 (Scatter Plot) & Phase 3 EDA Completion (Phase 3.6 & 4.4)**
> **Timestamp:** Monday, Nov 16, 2026 — 21:01 MYT (13:01 UTC)
> 
> *   **Visualization #4 Selection & Justification (Rubric 3c & 3d - Scatter Plot):** Constructed a multivariate Scatter Plot (`px.scatter`) mapping `icu_per_million` (X-axis) against `deaths_per_million` (Y-axis), color-coded by `location` with contextual hover metadata (`date`, `cases_per_million`, `vax_per_hundred`). *Selection Justification:* A scatter plot is the foundational bivariate visualization required to empirically validate linear and non-linear relationships between two continuous ratio variables ($r = 0.789$).
> *   **Empirical Discovery (Non-Linear Mortality Tipping Point):**
>     *   Below the `25/M` threshold, daily mortality across all four nations remained relatively contained (`< 5` deaths/million), demonstrating that unstrained ICUs successfully buffered severe cases.
>     *   Beyond the `25/M` Critical Overload line, mortality escalated non-linearly, peaking in **Malaysia on `2021-09-11`** (`17.44` deaths/million at `36.33` ICU patients/million, when vaccination was only at `116.11` doses/100 people during the Delta surge). This visually confirms that ICU saturation acts as a clinical tipping point for excess mortality.
> *   **Milestone Achieved:** All statistical and visual requirements of **Section 3c (Exploratory Data Analysis)** are now 100% complete.

> **Log Entry #13 — Visualizations #5 (Bar Chart) & #6 (Donut Chart) — Phase 4 Completion (Phase 4.5)**
> **Timestamp:** Monday, Nov 16, 2026 — 21:23 MYT (13:23 UTC)
> 
> *   **Semantic Color Encoding (Information Design):** Implemented a standardized clinical traffic-light palette (`#2ecc71` Safe Green, `#f1c40f` Caution Gold, `#e67e22` Warning Orange, `#e74c3c` Critical Red) across categorical visualizations to align visual perception with clinical risk severity.
> *   **Visualization #5 Selection & Justification (Rubric 3d - Bar Chart):** Constructed a Grouped Bar Chart (`px.bar`, `barmode='group'`, `text_auto=True`) comparing exact day counts across the 4 ICU stress regimes per country. *Selection Justification:* Grouped bar charts leverage pre-attentive length judgments along a common baseline, enabling immediate side-by-side comparison of discrete categorical counts (highlighting Israel's `164` and Malaysia's `138` critical days vs. Japan's `0` and South Korea's `3`).
> *   **Visualization #6 Selection & Justification (Rubric 3d - Pie/Donut Chart):** Constructed a Donut Chart (`px.pie`, `hole=0.45`) displaying the part-to-whole proportion of pandemic days across the 4 stress regimes (`39.5%` Low, `41.5%` Moderate, `10.7%` High, `8.33%` Critical). *Selection Justification:* Restricted strictly to 4 mutually exclusive categories summing to `100%`, where a center cutout reduces area-distortion bias while summarizing overall system resilience (`81.0%` of all country-days remained within Low/Moderate capacity).
> *   **Milestone Achieved:** All 6 required visualizations of **Section 3d (Data Visualization)** are now 100% complete.

> **Log Entry #14 — Dashboard Architecture, Interactive Filtering, & UI Professionalism (Phase 5.1)**
> **Timestamp:** Tuesday, Nov 17, 2026 — 12:48 MYT (04:48 UTC)
> 
> *   **Dashboard Framework Initialization (Rubric 3e):** Deployed the foundational Streamlit web application (`app.py`) backed by `@st.cache_data` memory caching for sub-second reactive filtering across all 3,660 records.
> *   **Interactive Filter Implementation (Rubric 3e):** Engineered three reactive sidebar controls (exceeding the minimum 2-filter rubric requirement): (1) Multi-country selector (`location`), (2) Chronological surveillance window slider (`date`), and (3) Analytical scale toggle (*Population-Adjusted Per Million* vs. *Raw Unadjusted Counts*).
> *   **Information Design & UI Audit:** Removed informal cartoon emojis (`🏥`, `🎛️`, `✅`) from the user interface. *Justification:* In clinical and academic decision-support systems, decorative emojis introduce visual noise and reduce institutional credibility. Adopted clean typographic hierarchy and structured layout framing.
