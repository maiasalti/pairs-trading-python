# Statistical Arbitrage: WTI vs. Brent Crude Oil Pairs Trading

## 1. Executive Summary
This project implements a quantitative trading strategy based on **Statistical Arbitrage** (Pairs Trading). It identifies and exploits pricing inefficiencies between two chemically similar but geographically distinct energy assets: **WTI Crude (CL)** and **Brent Crude (BZ)**.

Unlike simple correlation strategies, this system relies on **Cointegration**—a stricter statistical relationship implying a long-term mean-reverting spread. The strategy was rigorously developed using a Train/Test split methodology to eliminate look-ahead bias and validate performance on out-of-sample (unseen) data.

## 2. The Strategy Logic
The strategy operates on the economic principle of the "Law of One Price." Since WTI and Brent are both crude oil benchmarks, their prices are tethered by global supply and demand arbitrage. However, geopolitical events or local supply shocks create temporary price divergences.

* **Market Neutral:** By going Long one asset and Short the other, the strategy aims to remove exposure to the overall market direction (beta). It bets solely on the *spread* returning to equilibrium.
* **Mean Reversion:** Signals are generated when the spread stretches statistically significantly away from its historical mean (High Z-Score).

## 3. Technical Implementation

### Phase 1: Statistical Validation
* **Data Source:** Historical futures adjusted close data sourced via `yfinance`.
* **Hypothesis Testing:**
    * Initially tested equity pairs (e.g., PEP/KO) which showed high correlation (>0.8) but failed stationarity tests ($P > 0.05$).
    * Pivoted to WTI/Brent futures which confirmed a strong cointegrated relationship ($P < 0.05$).
* **Engle-Granger Test:** Used `statsmodels` to verify that the residuals of the pair were stationary, confirming a tradeable mean-reverting relationship.

### Phase 2: Signal Generation
* **Hedge Ratio ($\beta$):** Calculated using Ordinary Least Squares (OLS) regression on the **Training Set (2000-2010)** to avoid look-ahead bias.
    * Equation: $Price_{WTI} = \beta \times Price_{Brent} + Spread$
* **Z-Score Normalization:** The spread is normalized into standard deviations to create dynamic entry/exit signals.
    * **Entry:** Trade when $|Z| > 1.0$ (1 Standard Deviation).
    * **Exit:** Close when $|Z| < 0.0$ (Return to Mean).

### Phase 3: Backtesting & Validation
The system utilizes a vectorized backtest engine with an event-driven loop logic to manage trade state and position sizing.

* **Train/Test Split:**
    * **Training Data (2000-2010):** Used to calculate the Hedge Ratio, Mean, and Standard Deviation.
    * **Testing Data (2010-2015):** The fixed parameters were applied to this unseen period to simulate real-world performance.
* **Results:** The strategy remained profitable in the Out-of-Sample period, confirming the structural relationship held up without overfitting.
* **Risk Management:** Modeled transaction costs ($1.00 per round trip) to calculate realistic Net PnL.

## 4. Project Structure
* **Data Ingestion:** Automated downloading and cleaning of multi-asset time series.
* **Statistical Engine:** Implementation of ADF test, Cointegration test, and OLS regression.
* **Strategy Logic:** Calculation of spread dynamics and generation of Z-score signals.
* **Backtester:** A custom-built engine to simulate trade execution, PnL tracking, and equity curve plotting.

## 5. Technologies Used
* **Python 3.10+**
* **Pandas/NumPy:** Efficient time-series manipulation and vectorization.
* **Statsmodels:** Advanced statistical modeling (OLS, Cointegration).
* **Matplotlib:** Visualization of spread behavior and financial performance.
* **Yfinance:** Market data retrieval.

## 6. Key Learnings
1.  **Stationarity vs. Correlation:** Learned that high correlation is not sufficient for pairs trading; stationarity of the spread is the mathematical requirement for mean reversion.
2.  **Look-Ahead Bias:** The importance of calculating statistical parameters *only* on past data to prevent unrealistic backtest results.
3.  **Capital Efficiency:** By filtering for high-probability setups (High Z-Scores), the strategy minimizes commission drag compared to "always-in" strategies.

## 7. Future Improvements
* **Kalman Filter:** Implement a dynamic Hedge Ratio that updates in real-time to adapt to structural market changes (regime shifts).
* **Stop Loss Mechanism:** Add a volatility-based stop loss (e.g., close if Z > 4.0) to protect against "divergence risk" where the cointegration breaks permanently.
