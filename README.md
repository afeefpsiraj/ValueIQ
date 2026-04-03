# Indian Equity Valuation Tool

A web-based DCF valuation engine for NSE-listed Indian equities. Enter a stock ticker, and the tool automatically pulls financials, selects the appropriate valuation model, and computes intrinsic value with full sensitivity analysis.

![Python](https://img.shields.io/badge/Python-3.10+-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green) ![yFinance](https://img.shields.io/badge/yFinance-latest-orange)

---

## What it does

- Searches NSE-listed companies by name or ticker
- Fetches live financials from Yahoo Finance (revenue, operating income, PAT, capex, depreciation, working capital, debt, cash)
- Automatically detects financial vs non-financial companies and selects the appropriate model:
  - **FCFF** for non-financial companies (manufacturing, IT, FMCG, pharma, etc.)
  - **FCFE** for banks, NBFCs, insurance, and financial services
  - **Book Value fallback** when cash flow inputs fail sanity thresholds (e.g. FCFE > 50% upside for a bank)
- Computes **WACC** from actual balance sheet capital structure (not a hardcoded assumption)
- Uses **4-year median** for capex%, depreciation%, and NWC% ratios to smooth out one-off years
- Clamps growth rate to 2–15% and margins to realistic ranges to prevent model blow-up
- Outputs full **5-year projection tables** for both FCFF and FCFE models
- Generates **sensitivity analysis** across three dimensions: revenue growth rate, operating/net margin, and WACC

---

## Valuation Logic

### Model Selection

```
Is company financial (bank/NBFC/insurance)?
  → Yes: Use FCFE
       If FCFE upside > 50%: fallback to Book Value (prevents over-optimism)
  → No:  Use FCFF
       If all models (FCFF, FCFE, Book Value) < CMP: pick highest
       Else: FCFF → FCFE → Book Value in priority order
```

### FCFF Formula (Non-Financial)
```
FCFF = NOPAT + Depreciation - Capex - Change in NWC
NOPAT = Operating Income × (1 - Tax Rate)
Enterprise Value = PV of FCFFs + PV of Terminal Value
Equity Value = Enterprise Value - Debt + Cash
Intrinsic Value per Share = Equity Value / Shares Outstanding
```

### FCFE Formula (Financial)
```
FCFE = Net Income + Depreciation - Capex - Change in NWC
Equity Value = PV of FCFEs + PV of Terminal Value
Intrinsic Value per Share = Equity Value / Shares Outstanding
```

### WACC Calculation
```
WACC = (Debt/Total) × Cost of Debt + (Equity/Total) × Cost of Equity
Cost of Debt = 8% | Cost of Equity = 12%
Weights derived from actual balance sheet (Long Term Debt + Stockholders' Equity)
```

---

## Setup

**Requirements:** Python 3.10+

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/indian-equity-valuation.git
cd indian-equity-valuation

# Install dependencies
pip install -r requirements.txt

# Run
bash start.sh
# or: uvicorn app:app --host 0.0.0.0 --port 10000
```

Open `http://localhost:10000` in your browser.

---

## Stack

| Layer | Tech |
|---|---|
| Backend | Python, FastAPI |
| Valuation Engine | Custom DCF (FCFF/FCFE), Book Value |
| Data Source | yFinance (Yahoo Finance) |
| Stock Universe | NSE-listed companies (nse.csv) |
| Frontend | Vanilla HTML/CSS/JS |

---

## Limitations

- Data sourced from yFinance — accuracy depends on Yahoo Finance's data quality for Indian stocks
- Works best for companies with 3+ years of clean financials
- Not suitable for early-stage, loss-making, or recently-listed companies
- Terminal growth rate fixed at 4%; WACC cost assumptions are static defaults

---

## Why I built this

I wanted a tool that could run a proper DCF on Indian stocks without manually pulling numbers into a spreadsheet every time. Most free screeners give you ratios but not a bottom-up intrinsic value. This does the full calculation automatically including selecting the right model based on the company type so I can spend time on the thesis rather than the arithmetic.

---

## License

MIT
