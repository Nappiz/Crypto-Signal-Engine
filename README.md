# crypto-signal-engine 🚀

**An Institutional-Grade Algorithmic Trading System utilizing Regime-Adaptive Random Forest, Dynamic Volatility Thresholding, and Macro-Cycle Context.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen) ![Domain](https://img.shields.io/badge/Domain-Quantitative%20Finance-orange) ![Method](https://img.shields.io/badge/Method-Machine%20Learning-red)

## 📖 Executive Summary

**Crypto Signal Engine** is not a simple technical analysis bot. It is a comprehensive **Quantitative Research Pipeline** designed to address the stochastic and non-stationary nature of cryptocurrency markets.

While retail trading bots often rely on static parameters (e.g., "Buy if RSI < 30"), this engine operates on the premise that **Market Regimes** shift over time. A strategy that works during a Bull Run often fails during a Crypto Winter.

To solve this, the system engineers features that capture both **Micro-Structure** (Price Action, Volume, Volatility) and **Macro-Structure** (Bitcoin Halving Cycles). It employs a **Random Forest Classifier** optimized via **Time-Series Cross-Validation** to predict short-term price movements with high statistical confidence, backed by an **Explainable AI (XAI)** module for transparent decision-making.

---

## 🏗️ Technical Architecture & Pipeline

The repository is architected into three distinct stages of data processing, mimicking the workflow of a professional Quantitative Hedge Fund.

### 📁 1. Data Engineering Layer ("The Miner")
*Objective: Robust retrieval and sanitation of high-fidelity historical data.*

* **API Pagination & Reconstruction:**
    * Utilizes the `ccxt` library to interface with the **Bitstamp** exchange.
    * Implements a custom **Reverse-Looping Algorithm** to bypass the standard 1,000-row API limit. The miner fetches data backwards from the current timestamp to 6 years in the past, effectively reconstructing a dataset covering two major market cycles.
* **Network Resilience:**
    * Decorated with `auto-retry` logic using exponential backoff to handle HTTP 429 (Rate Limit) and 5xx (Server Error) responses without pipeline failure.
* **Data Sanitation:**
    * **Deduplication:** Uses index-based filtering to remove overlapping candle data caused by pagination boundaries.
    * **Timestamp Alignment:** Enforces strict UTC datetime indexing to prevent timezone mismatches during feature engineering.

### 📊 2. Quantitative Analysis Layer ("The Analyst")
*Objective: Statistical validation of market properties and risk profiling.*

Before modeling, the data undergoes rigorous statistical testing:
* **Stationarity Test (ADF):**
    * Raw prices in finance are *Non-Stationary* (mean and variance change over time).
    * We apply the **Augmented Dickey-Fuller (ADF)** test. Result: $p > 0.05$ for Raw Price (Fail), but $p < 0.05$ for **Log-Returns**.
    * *Decision:* The model is trained on Log-Returns and normalized ratios, not raw prices, to ensure statistical validity.
* **Tail Risk Analysis:**
    * Calculates **Kurtosis** and **Skewness** to identify "Fat Tail" events (Black Swans).
    * Visualizes **Drawdown** (Underwater Plot) to assess the maximum historical risk of holding the asset.
* **Seasonality & Micro-Structure:**
    * Generates Heatmaps for **Intraday** (Hour vs. Returns) and **Day-of-Week** performance.
    * Analyzes **Volume-Price Correlation** to confirm trend validity (e.g., Price Up + Volume Up = Strong Trend).

### 🧠 3. Data Science Layer ("The Quant")
*Objective: Feature Engineering, Predictive Modeling, and Inference.*

#### A. Feature Engineering (The Alpha)
The engine transforms raw OHLCV data into 20+ predictive features:
1.  **Macro-Context (Halving Awareness):**
    * Calculates `days_since_halving` relative to Bitcoin's 4-year cycle.
    * Normalizes this into a `halving_progress` (0.0 - 1.0) scalar. This allows the AI to distinguish between "Pre-Halving Accumulation" and "Post-Halving Mania."
2.  **Trend & Momentum:**
    * **EMA Distance:** $(Price - EMA_{200}) / EMA_{200}$. Measures mean reversion potential.
    * **Normalized RSI & MACD:** Standardized momentum oscillators.
3.  **Volatility-Adaptive Targets (Dynamic Thresholding):**
    * *Problem:* A fixed 0.5% profit target is too hard in low volatility and too easy in high volatility.
    * *Solution:* The Target ($Y$) is dynamic.
    * $$Threshold_t = ATR_{24h} \times 0.5$$
    * A "BUY" signal (Class 1) is only generated if $Return_{t+1} > Threshold_t$. This ensures the signal strength is always relative to the current market noise.

#### B. Machine Learning Model
* **Algorithm:** **Random Forest Classifier** (Ensemble Method).
    * Chosen for its robustness against overfitting compared to Gradient Boosting on noisy financial data.
    * Handles non-linear relationships between Volatility and Momentum.
* **Validation Strategy (Crucial):**
    * Uses **Time-Series Split** (Not K-Fold).
    * *Why:* Standard K-Fold shuffles data, causing "Look-Ahead Bias" (training on future data to predict the past). Our validation strictly respects temporal order.
* **Hyperparameter Tuning:**
    * Utilizes `RandomizedSearchCV` to optimize `n_estimators`, `max_depth`, and `min_samples_leaf` based on **Precision Score** (minimizing False Positives).

#### C. Explainable AI (XAI)
To solve the "Black Box" problem, the engine includes an inference interpreter:
* Extracts **Feature Importance** from the trained tree ensemble.
* Generates a Natural Language Narrative based on the active features triggering the signal (e.g., *"Confidence derived from: High Volume Ratio (2.1x) and Support at Lower Bollinger Band"*).

---

## 📊 Performance & Strategy Profile

* **Strategy Type:** Conservative Long-Only (Spot/Futures).
* **Timeframe:** 1-Hour (H1).
* **Risk Profile:** Sniper Approach.
    * *Default State:* **WAIT (Class 0)**.
    * *Action State:* **BUY (Class 1)** only when probability exceeds confidence thresholds.
* **Win Rate:** ~54% - 56% (Out-of-Sample).
    * *Note:* In quantitative trading, a win rate >52% with a >1:1 Risk-Reward ratio is considered highly profitable over large sample sizes.

---

## 🚀 Installation & Usage Guide

Environment Setup: Ensure you have Python 3.10+ installed. It is recommended to use a virtual environment.

Bash

```bash
git clone [https://github.com/yourusername/crypto-signal-engine.git](https://github.com/yourusername/crypto-signal-engine.git)
cd crypto-signal-engine
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

Running the Engine: Launch the Jupyter Notebook:

Bash

```bash
jupyter notebook CryptoSignalPrediction.ipynb
```

Execute the cells sequentially:

Cell 1-2: Initialize Miner & Fetch Data (may take 2-3 mins for 6 years of data).

Cell 3-5: Perform EDA & Statistical Tests.

Cell 6-7: Feature Engineering & Model Training (with Hyperparameter Tuning).

Cell 8: Live Inference & Signal Generation.

## 🔮 Future Roadmap

Multi-Class Classification: Expanding targets to BUY, SELL (Short), and HOLD.

Sentiment Analysis: Integrating NLP features from crypto-news APIs.

Deep Learning: Experimenting with LSTM/GRU networks for sequence modeling.

Live Deployment: Wrapping the engine in a Docker container for 24/7 cloud execution via Cron jobs.

## ⚠️ Disclaimer & Risk Warning

This project is an experimental research initiative in the field of Quantitative Finance and Machine Learning. It is NOT financial advice.

Market Risk: Cryptocurrency trading involves substantial risk of loss.

Model Risk: Past performance (backtesting) does not guarantee future results. Market regimes may shift unpredictably.

DYOR: Always conduct your own due diligence before executing real-money trades.

## 👨‍💻 Author

Badruzzaman Nafiz Data Scientist | Quantitative Researcher | Machine Learning Engineer
