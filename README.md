# Adaptive Portfolio Positioning under Market Volatility

**A Forward-Looking Regime-Switching Approach Using the VIX Term Structure**

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository contains the code and materials accompanying the paper:

> Liu, Chun Hin (2026). *Adaptive Portfolio Positioning under Market Volatility: A Forward-Looking Regime-Switching Approach Using the VIX Term Structure*. SSRN Working Paper.

The paper develops a practical dynamic risk-overlay framework that combines daily regime detection with weekly-scale execution constraints. A two-state Markov-switching GJR-GARCH model with time-varying transition probabilities (driven by lagged log(VIX) and the VIX term-structure inversion) generates forward-looking stress probabilities. These probabilities are mapped into continuous equity weights through a soft threshold rule, subject to a five-trading-day minimum holding period and stylised tax considerations.

Under strict expanding-window out-of-sample evaluation on SPY (2010–2026), the overlay reduces annualised volatility by approximately 25% and maximum drawdown by roughly six percentage points relative to buy-and-hold, while keeping turnover modest and preserving a comparable Sharpe ratio.

---

## Main Result (Two-Factor Model)

| Metric              | Strategy     | Buy & Hold  |
|---------------------|--------------|-------------|
| Mean Equity Weight  | ~0.77–0.80   | 1.00        |
| Ann. Return         | ~12.3%       | ~15.7%      |
| Ann. Volatility     | ~13.3%       | ~17.8%      |
| Max Drawdown        | ~−27.7%      | −33.7%      |
| Sharpe Ratio        | ~0.93        | ~0.88       |
| Annualised Turnover | ~1.56        | —           |

> Exact numbers may vary slightly across runs due to subsequent data revisions by the data provider and the stochastic nature of multi-start estimation. Qualitative conclusions are unaffected.

**Frozen execution parameters (locked):**
- Soft threshold: τ = 0.65
- Stress weight: W_STRESS = 0.50
- Minimum holding period: 5 trading days

---

## Repository Structure

```
├── README.md
├── paper/
│   └── Adaptive_Portfolio.pdf
├── notebooks/
│   ├── 01_Two_factor_main.ipynb          # Primary results (start here)
│   ├── 02_Three_factor_extension.ipynb   # Robustness extension
│   ├── 03_Weekly_baseline_ablation.ipynb
│   └── 04_Appendix_daily_3state_diagnosis.ipynb
├── results/                                # Key tables and figures (optional)
└── requirements.txt
```

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/<your-username>/adaptive-portfolio-vix-tvtp.git
cd adaptive-portfolio-vix-tvtp

# Create environment (recommended)
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the main notebook
jupyter notebook notebooks/01_Two_factor_main.ipynb
```

The main notebook downloads data via `yfinance` and reproduces the expanding-window out-of-sample results. Runtime is dominated by the multi-start maximum-likelihood estimation (typically 10–30 minutes depending on hardware).

---

## Model Summary

**Regime model**  
Two-state Markov-switching GJR-GARCH(1,1) with time-varying transition probabilities:

- State 0: Normal Growth  
- State 1: High-Volatility / Stress  

Transition drivers (all lagged one day):
- log(VIX)
- Inversion ratio = VIX / VIX3M

Economically motivated sign restrictions are imposed on stress-entry coefficients. Soft asymmetric persistence penalties encourage realistic regime durations.

**Position engine**  
Filtered stress probabilities are converted to equity weights via a soft threshold map:

```
w_t = 1 + (W_STRESS − 1) · clip( P(Stress)_t / τ , 0, 1 )
```

An (S,s)-style band and a five-trading-day minimum holding period further limit turnover. Transaction costs and a stylised short-term / long-term capital-gains tax are applied.

---

## Key Design Choices

- **Daily detection, weekly-scale execution** — regimes are identified every day, but positions change at most every five trading days.
- **Soft mapping is essential** — hard thresholds fail to reduce maximum drawdown because unconditional stress duration is short (~3.5 days).
- **Execution layer often binds** — sensitivity analysis shows that the probability-to-weight mapping is frequently more important for out-of-sample risk reduction than the precise transition coefficients.
- **Three-factor extension** (adding lagged Δlog(VIX)) does not systematically improve risk metrics or episode coverage and is retained only as robustness.

---

## Citation

If you use this code or find the paper useful, please cite:

```bibtex
@article{liu2026adaptive,
  title   = {Adaptive Portfolio Positioning under Market Volatility: A Forward-Looking Regime-Switching Approach Using the VIX Term Structure},
  author  = {Liu, Chun Hin},
  year    = {2026},
  note    = {SSRN Working Paper}
}
```

---

## License

This project is released under the MIT License. See `LICENSE` for details.

---

## Disclaimer

This repository is for academic and educational purposes only. It does not constitute investment advice. Past performance (including simulated out-of-sample results) is not indicative of future results. Users should conduct their own due diligence before making any investment decisions.
