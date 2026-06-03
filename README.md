# KRITI 2026: Long-Only Alpha Modeling in Equity Markets

This project implements a fully systematic, rules-based, long-only equity trading strategy for the Indian equity market. Developed for KRITI 2026, the system utilizes a Dual-Sleeve Hybrid Architecture tailored to navigate high-friction environments (0.536% round-trip transaction costs) while maximizing risk-adjusted returns.

## Executive Summary
The strategy bifurcates an initial capital of 5,000,000 INR into two distinct, uncorrelated engines: an IPO Penny Stock Momentum Sleeve and a multi-factor Convex Policy Optimization Sleeve. Over a 10-year backtest (2009-2020), the strategy achieved a 37.43% CAGR, significantly outperforming the Nifty 500 benchmark's 8.61%. It demonstrated superior capital preservation with a maximum drawdown of -25.08% compared to the benchmark's -31.82%.

---

## System Architecture

The core of the system is the **Dual-Sleeve** approach, dynamically balancing aggressive high-alpha plays with stable, factor-driven allocations.

* **Dynamic Capital Allocation:** By default, capital is split 30% to the IPO sleeve and 70% to the Core portfolio. This allocation shifts defensively to 10% IPO / 90% Core if the active stock universe exceeds 1,200 equities to minimize exposure to fragmented market noise.
* **IPO Penny Stock Momentum Sleeve:** Targets micro-cap assets with high price elasticity. It enforces a strict price ceiling of 40 INR and requires a 5-day post-listing observation window to ensure market equilibrium is established before capital is committed.
* **Core Portfolio Optimization Sleeve:** Utilizes a multi-factor ranking system based on a custom composite indicator blending momentum, volatility, and liquidity. It employs Multi-Period Optimization (MPO) to strictly penalize portfolio variance and transaction costs.

---

## Strategy Mechanics

### Core Portfolio: Feature Selection & Hysteresis
The stock selector evaluates the market cross-sectionally using normalized Z-scores bounded between -3 and 3. The custom composite score heavily favors momentum and liquidity while strictly penalizing volatility. 

To organically limit transaction costs and mitigate portfolio churn, the system employs a Hysteresis Filter. A stock enters the eligible universe if its rank is 65 or better, and is only removed once its rank drops to 79 or worse.

### Core Portfolio: Convex Optimization
Portfolio construction is framed as a distributionally robust Multi-Period Optimization problem. The allocation explicitly maximizes the worst-case conditional expected return while heavily penalizing quadratic dispersion and covariance instability. 

The core optimization objective targets:
w* = argmax { α·μᵀw - λ·E_μ(w) - γ·[Ψ(w) + η·E_Σ(w)] }

### IPO Sleeve: Risk Control & Queue System
* **Asset-Level Stop-Loss:** Triggers a liquidation if an asset's price retraces 20% from its High-Water Mark (HWM) since entry.
* **Sleeve-Level Drawdown:** If the total value of the IPO sleeve falls 30% from its peak, 50% of all active positions are liquidated and immediately reallocated to the Core portfolio. A 40% drawdown triggers a full exit.
* **Hierarchical Queue:** Vacated slots in the IPO portfolio are filled via a three-tier priority queue, sequentially prioritizing fresh listings, missed opportunities, and recovering assets that previously triggered a stop-loss.

---

## Performance Metrics (2009 - 2020)

The strategy displays a highly asymmetric capture profile, deliberately trading some market upside to severely restrict losses. The table below outlines the 10-year backtest performance against the Nifty 500 (CRSLDX) benchmark.

| Metric | Strategy | Nifty 500 |
| :--- | :--- | :--- |
| **CAGR** | 37.43% | 8.61% |
| **Annualized Volatility** | 15.05% | 17.39% |
| **Maximum Drawdown** | -25.08% | -31.82% |
| **Sharpe Ratio** | 1.947 | 0.633 |
| **Sortino Ratio** | 2.301 | N/A |
| **Information Ratio** | 1.274 | N/A |
| **Up-Capture** | 52.9% | 100% |
| **Down-Capture** | 19.6% | 100% |
| **Annualized Turnover** | 14.3x | N/A |

---

## Repository Overview & Execution

### Prerequisites
The codebase relies on standard Python quantitative finance and data science libraries. Ensure you have the following installed in your environment:
* `cvxpy`
* `cvxportfolio`
* `pandas`
* `numpy`
* `matplotlib`
* `yfinance` (for automatic benchmark fetching)
* `pyarrow` or `fastparquet` (for reading parquet data formats)

### Running the Backtest
The primary execution logic is contained within `2613_CodeBase_QuantFinChallenge.py`.

1. **Data Preparation:** Ensure the historical price dataset (e.g., `nse_prices_complete.parquet` or `.csv`) is located in the root directory. The script will automatically detect the file and align the necessary columns.
2. **Execution:** Run the codebase via your terminal:
   ```bash
   python 2613_CodeBase_QuantFinChallenge.py
   ```
3. **Simulation Workflow:** The script will progressively load the data, execute the IPO DD-liquidation strategy, run the feature selector, conduct the cvxportfolio multi-period backtest, combine the sleeves, and fetch the benchmark for comparison.
4. **Outputs:** Upon completion, the script generates a `strategy_output/` folder containing:
   * **Visualizations:** Equity Curve, Drawdown Curve, Portfolio Turnover, Position Count, and Rolling Outperformance charts.
   * **Logs:** Daily NAV tracking, trade logs, and detailed metrics summaries in CSV format.
   * **Report:** A comprehensive text file (`performance_report.txt`) summarizing the final statistics and rolling performance windows.
