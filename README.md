# NIFTY50 Market-Neutral Momentum Regimes: A Quantitative ML Pipeline

## Project Overview

This repository contains a systematic algorithmic trading pipeline designed to predict short-term, index-relative momentum breakouts across NIFTY50 stocks. The dataset used in this project and for the training of the model developed can be found here:

<https://huggingface.co/datasets/xxparthparekhxx/indian-stock-market-minute-data>

A major challenge with standard machine learning models in quantitative finance is the "Beta Illusion"—models often appear highly accurate simply by predicting upward movement during a broader market bull run. Furthermore, raw technical indicators scale with absolute price, making them non-comparable across different assets. This project solves these issues by transitioning from absolute price forecasting to **cross-sectional relative momentum**, neutralizing market beta to isolate idiosyncratic stock alpha.

## Core Architecture & Methodology

The objective of this pipeline is to identify **sustained, low-volatility momentum regimes** rather than erratic, news-driven price spikes. The architecture relies on three foundational pillars:

### 1. Market-Neutral Target Formulation

The binary target variable was redefined from a simple absolute return to a "sustained regime" target. A positive breakout (Class 1) is triggered only if a stock achieves positive forward returns _and_ its rolling volatility drops below its 90-day historical median. This explicitly filters out high-risk, volatile spikes in favor of stable trends.

### 2. Cross-Sectional Feature Normalization

To prevent the model from overfitting to static price levels or broad market rallies, technical features (MACD histograms, EMA percentage distances, Relative Volume) were engineered into **daily cross-sectional Z-scores**. The model evaluates how many standard deviations a stock's technical indicator deviates from the NIFTY50 market average on any specific day, allowing for dynamic, market-adjusted ranking.

### 3. Advanced Alpha Features

Beyond standard indicators, the pipeline incorporates academically-backed predictive features tailored for daily OHLCV data:

- **RSI:** Momentum indicator measuring the overbuying or underbuying of the given stock.
- **MACD:** Moving Average Convergence/Divergence (MACD) and its histogram is the ratio of EMA spanned across short-term 12 days and 26 days and identifies future possibilites of stocks and their sensitivity.
- **52-Week High Proximity:** Measuring momentum continuation.
- **Volume-Price Divergence:** Identifying anomalies where price drops but volume spikes (and vice versa) to flag structural reversals.
- **Sector-Relative Return Dispersion:** Measuring idiosyncratic strength against peer averages.

## Validation Strategy & Predictive Performance

Standard random train/test splits introduce severe look-ahead bias in financial time series. To prove the model's robustness, validation was conducted using a strict **Walk-Forward Time-Series Cross-Validation** loop.

The model (an XGBoost Classifier benchmarked against Logistic Regression) trained on expanding historical windows and tested strictly on subsequent unseen years.

**Key Results:**

- **Proven Stability:** The model successfully navigated distinct macroeconomic regimes (including market corrections and bull runs) without data leakage, yielding a highly defensible out-of-sample Mean ROC-AUC of ~0.52+.
- **Precision-Optimized:** Stripping away market beta lowered overall AUC but significantly improved **Class 1 Precision (up to 42%)**, indicating that when the model confidently signals a momentum breakout, it effectively identifies genuine relative strength.
- **Business Intelligence Integration:** The pipeline's daily predictions and historical validation metrics are exported to a custom 3-Page Power BI Dashboard, enabling end-users to screen daily breakout probabilities and assess historical risk-adjusted win rates.

## Model Limitations & Analytical Caveats

While this model demonstrates a statistically significant edge in identifying sustained momentum regimes, it is essential to acknowledge the realities of applying machine learning to highly efficient financial markets:

1. **The Technical Indicator Ceiling:** This model relies on daily OHLCV data and standard technical indicators. In highly liquid, heavily institutionalized large-cap markets like the Nifty 50, standard technical setups are rapidly arbitraged by algorithmic trading. A ROC-AUC in the 0.52 - 0.58 range is the realistic ceiling for this specific feature space; any higher would likely indicate data leakage rather than genuine alpha.
2. **Target Formulation Bias:** By defining the target as a positive return _combined_ with a drop in rolling volatility, the model intentionally filters out explosive but erratic breakouts. While safer for risk-adjusted portfolio management, it means the model will naturally miss sudden, news-driven price spikes.
3. **Frictionless Backtesting:** The current validation metrics do not account for real-world trading frictions, including slippage, bid-ask spreads, and capital gains taxes. A statistically significant predictive edge does not automatically guarantee a profitable trading strategy once execution costs are applied.

---

_Note: This project is for educational and analytical purposes only. It does not constitute financial advice._
