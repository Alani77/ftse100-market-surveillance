# Untitled

# FTSE 100 Market Surveillance & Econometric Volatility Pipeline

An automated market surveillance and time-series risk pipeline built in Python. This project ingests 5 years of daily price and volume data across 15 liquid FTSE 100 equities, establishes rolling statistical baselines to detect abnormal trading activity, and estimates time-varying conditional volatility using an **AR(1)-GARCH(1,1)** framework.

## Project Motivation

Capital market regulators and risk teams monitor live and historical order flow to identify potential insider trading, market manipulation, and structural liquidity shocks. This pipeline automates first-line surveillance by flagging days where an equity experiences a simultaneous, statistically significant price jump and volume surge, while accounting for volatility clustering.

## Methodology

### 1. Data Ingestion & Preprocessing

- **Universe:** 15 major FTSE 100 constituents across Financials, Energy, Healthcare, Consumer Goods, and Mining (`HSBA.L`, `BARC.L`, `LLOY.L`, `NWG.L`, `SHEL.L`, `BP.L`, `AZN.L`, `GSK.L`, `ULVR.L`, `DGE.L`, `RIO.L`, `GLEN.L`, `TSCO.L`, `VOD.L`, `LSEG.L`).
- **Source:** 5 years of daily adjusted Close prices and Trading Volume via the Yahoo Finance API (`yfinance`).
- **Returns:** Continuously compounded daily log returns:
    
    $$
    r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
    $$
    

### 2. Dual-Threshold Anomaly Detection

To isolate genuine market events from routine noise, each trading day is evaluated against a trailing 30-day rolling window:

- **Return Z-Score:** $Z_{r,t} = \frac{r_t - \mu_{r,30}}{\sigma_{r,30}}$
- **Volume Z-Score:** $Z_{v,t} = \frac{V_t - \mu_{v,30}}{\sigma_{v,30}}$
- **Surveillance Flag Rule:** A high-priority alert is triggered when $|Z_{r,t}| \ge 2.5$ **and** $Z_{v,t} \ge 2.5$ (a $\ge 2.5\sigma$ price movement accompanied by a $\ge 2.5\sigma$ volume spike).

### 3. Econometric Volatility Modeling: AR(1)-GARCH(1,1)

Financial asset returns exhibit volatility clustering (leptokurtic residuals and time-varying variance). To model conditional heteroskedasticity, the pipeline fits an **AR(1)-GARCH(1,1)** specification using Maximum Likelihood Estimation (`arch`):

- **Mean Equation:** $r_t = c + \phi_1 r_{t-1} + \epsilon_t$, where $\epsilon_t = \sigma_t z_t$ and $z_t \sim \mathcal{N}(0,1)$
- **Variance Equation:** $\sigma_t^2 = \omega + \alpha_1 \epsilon_{t-1}^2 + \beta_1 \sigma_{t-1}^2$

## Visual Diagnostics

*Figure 1: Daily log returns for Barclays (`BARC.L`) overlaid with $\pm 2\sigma$ GARCH(1,1) conditional volatility bands and flagged surveillance anomalies (top), paired with the 30-day rolling volume Z-score against the $2.5\sigma$ alert threshold (bottom).*

![surveillance_diagnostics.png](Untitled/surveillance_diagnostics.png)

## Repository Structure

- `surveillance_pipeline.py` — End-to-end Python script for data ingestion, anomaly scoring, GARCH estimation, and plotting.
- `flagged_trading_anomalies.csv` — Automated audit log of all flagged ticker-date anomalies sorted chronologically.
- `surveillance_diagnostics.png` — High-resolution diagnostic chart comparing flagged events against conditional volatility.

## How to Run Locally

1. Clone the repository:
    
    ```bash
    git clone [<https://github.com/Alani77/ftse100-market-surveillance.git>](<https://github.com/Alani77/ftse100-market-surveillance.git>)
    cd ftse100-market-surveillance
    ```
    
2. Install dependencies:
    
    ```bash
    pip install yfinance pandas numpy statsmodels arch matplotlib
    ```
    
3. Run the surveillance pipeline:
    
    ```bash
    python surveillance_pipeline.py
    ```
    

## Author

**Mustapha Ganiyu**
