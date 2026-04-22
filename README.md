# Merton_credit_risk_R
Merton structural credit risk model implemented in R. Estimates probability of default for S&amp;P 500 firms via numerical optimization of implied asset value and volatility, with sensitivity analysis and interactive 3D visualization.
# Merton Credit Risk Model — S&P 500

An end-to-end implementation of the **Merton structural credit risk model** in R, applied to any publicly traded firm in the S&P 500. The algorithm automates the full pipeline: from market data retrieval to probability of default estimation, sensitivity analysis, and interactive visualizations.

---

## Overview

The Merton model treats a firm's equity as a **call option on its assets**, allowing us to back out the implied asset value and asset volatility from observable market data. From these, we derive the **Probability of Default (PD)** — a core metric in credit risk management and Basel II/III frameworks.

This project solves the Merton system numerically by jointly minimizing the two structural equations (24.3 and 24.4) via numerical optimization, then extends the analysis with sensitivity testing and 3D visualization of the objective function surface.

---

## Methodology

### Merton's Structural Equations

The model relies on two simultaneous equations:

**Equation 1 — Equity as a call option (Black-Scholes):**

$$E_0 = V_0 \cdot N(d_1) - D \cdot e^{-r_f T} \cdot N(d_2)$$

**Equation 2 — Equity-asset volatility relationship:**

$$\sigma_E \cdot E_0 = N(d_1) \cdot \sigma_V \cdot V_0$$

Where:

$$d_1 = \frac{\ln(V_0/D) + (r_f + \sigma_V^2/2) \cdot T}{\sigma_V \sqrt{T}}, \quad d_2 = d_1 - \sigma_V \sqrt{T}$$

The system is solved by minimizing $F(V_0, \sigma_V)^2 + G(V_0, \sigma_V)^2$ using `optim()`.

### Probability of Default

$$PD = N(-d_2) = N\left(-\frac{\ln(V_0/D) + (r_f + \sigma_V^2/2) \cdot T}{\sigma_V \sqrt{T}} + \sigma_V \sqrt{T}\right)$$

### Additional Credit Metrics

| Metric | Description |
|---|---|
| Market Value of Debt | $V_0 - E_0$ |
| PV of Promised Payment | $D \cdot e^{-r_f T}$ |
| Expected Loss | $(PV\_debt - MV\_debt) / PV\_debt$ |
| Recovery Rate | $1 - EL / PD$ |

---

## Features

- **Automated data retrieval** — pulls 3 years of adjusted closing prices from Yahoo Finance via `quantmod`
- **S&P 500 company lookup** — matches ticker to market cap from a structured dataset of 500+ firms
- **Numerical optimization** — jointly solves for implied asset value ($V_0$) and asset volatility ($\sigma_V$)
- **PD estimation** — calculates risk-neutral probability of default at any maturity
- **Sensitivity analysis** — evaluates PD across a grid of debt values and maturities ($\pm 50\%$)
- **3D surface visualization** — interactive Plotly plot of the objective function over $(V_0, \sigma_V)$ space
- **Contour plot** — highlights the global minimum, confirming solution stability

---

## Inputs

The user provides three inputs at runtime:

| Input | Description | Example |
|---|---|---|
| `Ticker` | Stock ticker symbol | `AAPL` |
| `Maturity (T)` | Time horizon for default (years) | `1` |
| `Debt (D)` | Total face value of debt | `110B` |

Debt values can be entered in millions (`M`), billions (`B`), or trillions (`T`). Market cap is retrieved automatically from the S&P 500 dataset.

> **Note:** The risk-free rate (`rf`) should be updated to the current 1-year Treasury yield before running. It is defined in Step 8 of the notebook.

---

## Outputs

```
Probability of default at 1 years is: 2.34 %
```

```
| Parameter              | Value     |
|------------------------|-----------|
| Market Value of Assets | 182.4 B   |
| Asset Volatility       | 0.2341    |
| Market Value of Debt   | 94.7 B    |
| Expected Loss          | 0.0187    |
| Recovery Rate          | 0.8923    |
| Probability of Default | 0.0234    |
```

Plus interactive 3D surface and contour plots of the optimization landscape.

---

## Project Structure

```
merton-credit-risk-r/
│
├── merton_credit_risk.ipynb   # Main notebook (20 steps, fully documented)
├── sp500_companies.csv        # S&P 500 company data with market caps
└── README.md
```

---

## Requirements

```r
install.packages(c("quantmod", "plotly", "tidyverse", "readr", "knitr",
                   "Sim.DiffProc", "ggplot2", "dplyr", "pROC"))
```

Tested on R 4.3+. Requires an active internet connection for Yahoo Finance data retrieval.

---

## Limitations & Extensions

- The Merton model assumes a **simple capital structure** (single zero-coupon debt obligation). Real firms have complex debt schedules.
- Asset value follows **geometric Brownian motion** — fat tails and jumps are not captured.
- The risk-free rate is **manually set** and should reflect the current market rate for the chosen maturity.

Potential extensions:
- Integration with FRED API for automatic risk-free rate retrieval
- Multi-firm batch analysis across S&P 500 sectors
- Comparison with KMV/EDF approach (distance-to-default)
- CDS spread implied PD validation

---

## References

- Merton, R.C. (1974). *On the Pricing of Corporate Debt: The Risk Structure of Interest Rates.* Journal of Finance, 29(2), 449–470.
- Hull, J.C. (2018). *Options, Futures, and Other Derivatives* (10th ed.). Chapter 24.
- Basel Committee on Banking Supervision. (2006). *International Convergence of Capital Measurement and Capital Standards.*
