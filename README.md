# Statistical Arbitrage: Paired Switching Strategy (WTI vs. Brent Crude)

## 1. Project Overview
This project implements an algorithmic trading strategy based on **Statistical Arbitrage** (Pairs Trading). It identifies and exploits pricing inefficiencies between two cointegrated assets: **WTI Crude Oil (CL)** and **Brent Crude Oil (BRN)**.

Unlike standard correlation strategies, this system relies on **Cointegration**—a stricter statistical relationship implying a long-term mean-reverting spread. The algorithm was backtested over a 3-year period (2020–2023), demonstrating profitability even after accounting for transaction costs.

## 2. The Strategy Logic
The strategy follows a classic Mean Reversion approach:
1.  **Selection:** Confirmed cointegration between WTI and Brent using the **Engle-Granger Two-Step Test** (P-value < 0.05).
2.  **Hedge Ratio:** Calculated the dynamic hedge ratio using **OLS (Ordinary Least Squares) Regression** to construct a market-neutral spread.
    * *Formula:* $Spread = Price_{WTI} - (\beta \times Price_{Brent})$
3.  **Signal Generation:** Normalized the spread into a **Z-Score** to identify statistical anomalies.
    * **Short Signal:** Z-Score > 1.0 (Spread is too wide).
    * **Long Signal:** Z-Score < -1.0 (Spread is too narrow).
    * **Exit:** Z-Score reverts to 0.0 (Mean).

## 3. Tech Stack
* **Python 3.10+**
* **yfinance:** Data ingestion.
* **Statsmodels:** Engle-Granger test, OLS Regression, ADF test.
* **Pandas/NumPy:** Vectorized backtesting and data manipulation.
* **Matplotlib:** Visualization of equity curves and spread dynamics.

## 4. Performance Metrics (2020–2023)
* **Total Trades:** 26 (High selectivity/discipline)
* **Gross Profit:** $74.13 (Per single unit pair)
* **Net Profit:** $48.13 (After transaction costs)
* **Transaction Costs:** modeled at $1.00 per round-trip trade.
* **Outcome:** The strategy remained profitable despite high market volatility (Covid-19, Geopolitical conflict), validating the "Market Neutral" hypothesis.

## 5. Key Learnings
* **Stationarity vs. Correlation:** Learned that high correlation does not imply cointegration; stationarity of the spread is the key driver of mean reversion.
* **Lookahead Bias:** Implemented careful indexing to ensure the backtest did not use future data for trade decisions.
* **Capital Efficiency:** The strategy demonstrated that minimizing trade frequency (trading only high-probability Z-scores) significantly improves Net PnL by reducing commission drag.

## 6. Future Improvements
* Implement a **Kalman Filter** to dynamically update the Hedge Ratio over time (instead of a static OLS ratio).
* Add a **Stop-Loss** mechanism to protect against "regime changes" where the cointegration breaks permanently.
