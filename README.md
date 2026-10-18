# AeroPulse
Modular METAR parser and deep-learning pipeline for time-series aviation weather nowcasting, predicting the next hour's METAR parameters directly from historical observation sequences.

> A modular aviation weather toolkit that parses raw, unstructured METAR reports into clean tabular datasets and uses time-series machine learning models to forecast next-hour (t+1) weather observations and flight categories (VFR / MVFR / IFR / LIFR).

> ⚠️ **Project Status: Active Development**  
> The deterministic METAR decoder is fully functional. The tabular time-series feature engineering and ML forecasting models are actively being trained, evaluated, and tuned.

## Project Preview- 
<img width="992" height="930" alt="image" src="https://github.com/user-attachments/assets/7351daff-2ac4-46c5-899c-6e93f4b38538" />

## Why This Project?

Aviation safety and dispatch decisions rely heavily on **METAR** (Meteorological Aerodrome Report) observations:
1. **Raw reports are hard to parse:** Encoded tokens, international notation differences, variable remarks (`RMK`), and unscheduled updates (`SPECI`) make parsing tricky without dedicated cleaning.
2. **Current conditions change quickly:** Pilots and dispatchers frequently rely on the latest report, assuming persistence. Rapid convective development or shifting fog banks can quickly push an airfield below landing minimums.

**AeroPulse** bridges this gap: it turns messy string tokens into structured time-series data, generates lag features, and trains machine learning models to anticipate the **next observation state (t+1)** before it is published.

## Key Features

### 1. Deterministic METAR Decoder
- **Full Field Parsing:** Decodes station ICAO, observation timestamp, wind speed/direction/gusts, prevailing visibility, runway visual range (RVR), weather phenomena (e.g., `TSRA`, `BR`, `-SN`), cloud cover/bases, temperature, dew point, and altimeter setting (`QNH`).
- **Feature Derivation:** Automatically calculates flight rule categories (`VFR`, `MVFR`, `IFR`, `LIFR`), dew-point depression, and decomposes cyclical wind angles into continuous $u$ and $v$ vectors.
- **Clean Schema:** Emits structured Python dictionaries or Pandas DataFrames ready for analysis.

### 2. Time-Series Machine Learning Engine
- **Lag Feature Engineering:** Creates rolling temporal features (e.g., t-1, t-2, t-3 hour moving averages, rate of pressure change) to capture atmospheric trends without unnecessary deep learning complexity.
- **Dual Forecasting Tasks:**
  - **Regression:** Predicts continuous values for the next hour (temperature, dew point, altimeter, lowest cloud base).
  - **Classification:** Predicts whether the station will hold or transition between flight categories (e.g., VFR to IFR).
- **Interpretable Models:** Uses standard Scikit-Learn models and XGBoost to prioritize fast training, easy debugging, and high interpretability.

  ## Tech Stack

| Layer | Tools & Libraries | Purpose |
| :--- | :--- | :--- |
| **Core & Parsing** | `Python 3.10+` `re` `Dataclasses` | Deterministic token extraction and schema definitions |
| **Data & Feature Engineering** | `Pandas` `NumPy` | Data cleaning, time-series alignment, and rolling lag features |
| **Machine Learning** | `Scikit-Learn` `XGBoost` | Baseline models, regression, and classification |
| **Evaluation & Plotting** | `Matplotlib` `Seaborn` | Actual vs. predicted error curves and confusion matrices |
| **Testing** | `Pytest` | Edge-case unit tests for malformed METAR strings |

## Project Roadmap
This roadmap outlines the development phases of **AeroPulse**, from its deterministic aviation weather decoding engine to multi-airport time-series nowcasting, deep learning models, and real-time operational deployment.
### Phase Overview & Status
| Phase | Title | Focus Area | Status |
| :---: | :--- | :--- | :---: |
| **01** | **Deterministic METAR Engine** | Core regex token parser, flight rules (VFR/IFR), live API fetcher | `Completed`  |
| **02** | **Tabular Feature Engineering & ML** | 6h lag matrices, pressure trends, multi-airport XGBoost models | `Active Development`  |
| **03** | **Multi-Horizon & Deep Learning** | $t+1$ to $t+6$ forecasts, LSTM/Transformers, uncertainty bounds | `Next Up`  |
| **04** | **Spatial Context & Multi-Modal Ingestion** | Regional airport networks, TAF comparison, gridded NWP data | `Planned`  |
| **05** | **Production API & Pilot Dashboard** | FastAPI streaming backend, interactive map UI, Dockerized workers | `Planned`  |
| **06** | **PyPI Package & Ecosystem** | `pip install aeropulse`, CLI tools, open-source benchmarks | `Future Vision`  |
---
### Phase 1: Deterministic METAR Engine & Core Normalization
**Status:** `Completed`   
*Goal: Build an industrial-grade, regex-based meteorological decoder that turns raw, unstructured METAR strings into structured, standardized aviation domain objects.*
- [x] **Deterministic METAR Parser (`parser.py`)**:
  - [x] Full regex token extraction for wind (direction, speed, gusts, variable headings).
  - [x] Cloud layer decomposition (FEW, SCT, BKN, OVC, VV) and lowest ceiling detection.
  - [x] Temperature, dew point, and altimeter setting ($QNH$ in hPa / inHg).
  - [x] Metric and statute mile visibility normalization.
  - [x] Weather phenomena categorization (fog, rain, snow, thunderstorms, haze).
- [x] **Aviation Domain Logic**:
  - [x] Automatic flight category derivation (`VFR`, `MVFR`, `IFR`, `LIFR`).
  - [x] Dew point depression calculation (fog condensation metric).
  - [x] Continuous wind vector decomposition into orthogonal $u$ and $v$ components.
- [x] **Live Ingestion & Global Airport Resolution**:
  - [x] Dual-source live fetching (NOAA Aviation Weather Center + CheckWX API).
  - [x] Worldwide ICAO (4-letter) and IATA (3-letter) code resolver with offline fallback.
- [x] **Test Coverage**:
  - [x] Edge-case unit tests for malformed international reports (`test_parser.py`).
---
### Phase 2: Tabular Feature Engineering & Baseline ML Nowcaster
**Status:** `Active Development`  
*Goal: Construct time-series lag datasets from historical archives and establish benchmark machine learning models for 1-hour ($t+1$) nowcasting.*
- [x] **Historical Data Pipeline (`dataset_fetcher.py`)**:
  - [x] Automated batch downloading of multi-year METAR archives from the NOAA / IEM ASOS network.
  - [x] Multi-climatology sampling (KJFK - coastal/temperate, VIDP - subtropical fog, EGLL - maritime).
- [x] **Atmospheric Time-Series Feature Engineering (`feature_engineering.py`)**:
  - [x] Hourly temporal regularization and short-gap time interpolation.
  - [x] Diurnal solar cycle embedding ($\sin/\cos$ 24-hour cyclical transforms).
  - [x] 6-hour sliding window lag generation ($t_0$ down to $t_{-5}$).
  - [x] Atmospheric tendency features (3-hour altimeter pressure trend $\Delta P$, dew point depression delta).
  - [x] Rolling 6-hour visibility statistics (minimum and moving average).
- [x] **Machine Learning Pipeline (`train.py`)**:
  - [x] Chronological train/validation/test splits to eliminate temporal data leakage.
  - [x] Aviation gold standard baseline: **Persistence Benchmark** ($t+1 = t_0$).
  - [x] XGBoost regression models for next-hour visibility and lowest cloud ceiling.
  - [x] Automated serialization of model weights and feature schemas into `models/`.
- [x] **Inference & Visualization (`predict.py`, `evaluate.py`)**:
  - [x] `MetarNowcaster` sequence inference class.
  - [x] 72-hour actual vs. predicted tracking curves and top-10 atmospheric feature importance plots.
- [ ] **Phase Refinement**:
  - [ ] Hyperparameter tuning (Bayesian optimization / Optuna).
  - [ ] Support for missing observation flags and automated data validation schemas.
---
### Phase 3: Multi-Horizon Forecasting & Deep Learning Architectures
**Status:** `Next Up`  
*Goal: Expand beyond next-hour predictions to multi-step forecasts ($t+1$ to $t+6$) and benchmark sequential deep learning architectures against gradient-boosted trees.*
- [ ] **Multi-Step Forecasting Horizons**:
  - [ ] Extended horizons: nowcast predictions for $t+1$, $t+2$, $t+3$, and $t+6$ hours.
  - [ ] Direct multi-output regression vs. autoregressive iterative rollout.
- [ ] **Sequential Deep Learning Models**:
  - [ ] Recurrent baselines (LSTM / GRU) tailored for unevenly spaced observation sequences.
  - [ ] Temporal Convolutional Networks (TCN) and Temporal Fusion Transformers (TFT).
  - [ ] Cross-model benchmark table comparing XGBoost, LightGBM, CatBoost, and LSTM/Transformer.
- [ ] **Direct Flight Category Classification**:
  - [ ] Dedicated classification models predicting transition probabilities (e.g., $P(\text{VFR} \to \text{IFR})$ within 3 hours).
  - [ ] Class-weighted loss functions to handle severe weather class imbalances (rare fog and thunderstorm events).
- [ ] **Uncertainty & Confidence Intervals**:
  - [ ] Quantile regression ($10^{\text{th}}$, $50^{\text{th}}$, $90^{\text{th}}$ percentiles) for ceiling and visibility.
  - [ ] Conformal prediction bounds for flight planning safety buffers.
---
### Phase 4: Spatial Context & Multi-Modal Meteorological Ingestion
**Status:** `Planned`   
*Goal: Incorporate spatial weather fronts by ingesting regional airport clusters and fusing external numerical forecasts.*
- [ ] **Spatial Airport Networks (Graph / Multi-Station Ingestion)**:
  - [ ] Spatial lag features from upstream/neighboring airfields (e.g., tracking a frontal system moving along the US Northeast corridor or Northern India).
  - [ ] Graph Neural Network (GNN) prototype for airport cluster nowcasting.
- [ ] **TAF (Terminal Aerodrome Forecast) Integration**:
  - [ ] Parser for human-authored TAF forecasts.
  - [ ] Automated validation and comparison: AeroPulse ML Nowcasts vs. Official TAF forecasts.
- [ ] **Gridded NWP & Satellite Data**:
  - [ ] Ingestion of Numerical Weather Prediction (NWP) outputs (e.g., NOAA HRRR / GFS model soundings).
  - [ ] Integration of regional radar reflectivity metrics for convective storm tracking.
---
### Phase 5: Production Service, API & Pilot Dashboard
**Status:** `Planned`   
*Goal: Package AeroPulse as an operational real-time service with streaming APIs and an interactive visual dashboard.*
- [ ] **High-Performance REST & Streaming API**:
  - [ ] FastAPI backend delivering real-time predictions, airport lookups, and historical backtests.
  - [ ] OpenAPI / Swagger documentation and health check endpoints.
  - [ ] WebSocket streaming for live airfield condition monitors.
- [ ] **Interactive Web Dashboard**:
  - [ ] Interactive UI (Streamlit or Next.js + Leaflet/Cesium) displaying live airport maps.
  - [ ] 6-hour historical trend charts alongside projected cone-of-uncertainty nowcasts.
  - [ ] Flight rule alert triggers when conditions are projected to breach landing minimums.
- [ ] **Containerization & MLOps**:
  - [ ] Docker containerization for one-command deployment.
  - [ ] Automated background cron worker for continuous live observation ingestion.
  - [ ] Model tracking and experiment logging (MLflow / Weights & Biases).
---
### Phase 6: Python Package Release & Community Ecosystem
**Status:** `Future Vision`   
*Goal: Distribute AeroPulse as a battle-tested Python library for the aviation and meteorological open-source communities.*
- [ ] **PyPI Library Release**:
  - [ ] Clean modular architecture installable via `pip install aeropulse`.
  - [ ] Decoupled modules: `aeropulse.parser`, `aeropulse.features`, `aeropulse.models`.
- [ ] **Command-Line Interface (CLI)**:
  - [ ] `aeropulse fetch <ICAO>` - Instant live decoding with human-readable flight rules.
  - [ ] `aeropulse forecast <ICAO>` - Run nowcast directly from the terminal.
- [ ] **Comprehensive Documentation & CI/CD**:
  - [ ] GitHub Actions CI running automated pytest suites, linting, and type checks on every PR.
  - [ ] Interactive Jupyter demo notebooks for researchers and flight operations teams.

## Author & Contact

Developed by **Sohi Kulkarni**

[![GitHub](https://img.shields.io/badge/GitHub-@codewithsohi-181717?style=flat-square&logo=github)](https://github.com/codewithsohi)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/sohi-kulkarni)
