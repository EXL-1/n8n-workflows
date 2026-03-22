# 📈 Stock Market Monitor — Oversold & Overbought Screener

> An automated n8n workflow that screens 258 stocks across US, EU, and UK
> markets daily, calculates 180-day SMA deviation and RSI-14, generates an
> AI-powered investment summary via Google Gemini, and delivers a formatted
> HTML report to your inbox every weekday morning.

[![n8n](https://img.shields.io/badge/n8n-workflow-orange?logo=n8n)](https://n8n.io)
[![Gemini AI](https://img.shields.io/badge/Google-Gemini%20AI-blue?logo=google)](https://ai.google.dev)
[![Data](https://img.shields.io/badge/Data-Yahoo%20Finance%20API-purple)](https://finance.yahoo.com)
[![Schedule](https://img.shields.io/badge/Schedule-8AM%20Weekdays-green)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🧭 Why This Exists

Manually scanning hundreds of stocks every morning for technical
opportunities is slow, inconsistent, and easy to get wrong. This workflow
automates the entire pipeline — data fetching, indicator calculation,
AI analysis, and delivery — so you get a structured, actionable report
waiting in your inbox before the market opens.

---

## ⚡ Quick Start

### 1. Import the workflow into n8n

```bash
# In the n8n UI:
# Settings → Import Workflow → Upload file
```

Upload `Stock Market Monitor - Gemini AI + Gmail.json` from this repository.

### 2. Configure your credentials

| Credential | Node | Where to get it |
|---|---|---|
| Google Gemini API | `Google Gemini Chat Model` | [Google AI Studio](https://aistudio.google.com) |
| Gmail OAuth2 | `Send Gmail Report` | [Google Cloud Console](https://console.cloud.google.com) |

### 3. Update recipient addresses

Open the `Send Gmail Report` node and edit the **Send To** field:

```
your@email.com
```

### 4. Activate the workflow

Toggle **Active** in the top-right corner of the n8n editor.
The workflow will now run automatically at **8:00 AM every weekday**.

---

## 🗺️ Workflow Architecture

```
┌─────────────────────────┐
│  Daily Schedule (8AM)   │  Triggers Mon–Fri via cron: 0 8 * * 1-5
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Fetch All Stocks &     │  Calls Yahoo Finance v8 Chart API for
│  Calculate              │  258 tickers. Computes 180d SMA,
│                         │  RSI-14, deviation %, dividend yield,
│                         │  and ex-dividend dates.
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Format HTML Email &    │  Builds styled HTML tables (oversold /
│  AI Input               │  overbought per market). Also formats
│                         │  a plain-text summary for the AI prompt.
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐     ┌──────────────────────────┐
│  AI Investment Analysis │◄────│  Google Gemini Chat Model │
│  (LangChain Agent)      │     │  gemini-flash-latest      │
└───────────┬─────────────┘     └──────────────────────────┘
            │
            ▼
┌─────────────────────────┐
│  Combine AI Summary +   │  Merges the Gemini analysis section
│  Report                 │  with the full HTML screener tables.
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Send Gmail Report      │  Delivers the complete HTML email
│                         │  to all configured recipients.
└─────────────────────────┘
```

---

## 📊 What the Report Contains

Each daily email contains two sections.

### Section 1 — AI Investment Analysis (Google Gemini)

A structured HTML summary with:

- **Executive Summary** — 2–3 sentence market overview
- **Top 5 Oversold Picks** — Value opportunities with specific reasoning
- **Dividend Opportunities** — Oversold stocks with attractive yields
- **Caution: Overbought Stocks** — Stocks flagged for potential pullback
- **Sector Themes** — Which sectors are stretched in either direction
- **Risk Assessment** — Anomalies, RSI divergences, data warnings
- **Bottom Line** — 3–4 actionable takeaways

### Section 2 — Global Stock Screener Tables

Three markets, each showing:

| Column | Description |
|---|---|
| **Ticker** | Stock symbol |
| **Name** | Full company name |
| **Price** | Latest closing price |
| **180d SMA** | 180-day Simple Moving Average |
| **Dev %** | % deviation from 180d SMA (negative = below, positive = above) |
| **RSI** | 14-period Relative Strength Index (bold if < 30 or > 70) |
| **Signal** | `OVERSOLD` / `OVERBOUGHT` / `NEUTRAL` |
| **Ex-Dividend** | Most recent ex-div date; highlighted if within last 30 days |
| **Yield** | Estimated annualized dividend yield |

#### Signal thresholds

```
OVERSOLD   → Deviation < -3%
OVERBOUGHT → Deviation > +3%
NEUTRAL    → -3% ≤ Deviation ≤ +3%

RSI alert  → Bold red if RSI < 30 (momentum oversold)
           → Bold orange if RSI > 70 (momentum overbought)
```

#### Markets covered

| Market | Universe | Tickers |
|---|---|---|
| 🇺🇸 United States | S&P 500 | ~130 tickers |
| 🇪🇺 European Markets | Euro Stoxx / Major EU | ~65 tickers |
| 🇬🇧 United Kingdom | FTSE 100 | ~63 tickers |

---

## 🛠️ Prerequisites

Before importing this workflow you need:

- [ ] **n8n** self-hosted or cloud (v1.0+)
  → [Install n8n](https://docs.n8n.io/hosting/)
- [ ] **Google Gemini API key**
  → [Get a free key at Google AI Studio](https://aistudio.google.com/app/apikey)
- [ ] **Gmail account** with OAuth2 credentials configured in n8n
  → [n8n Gmail setup guide](https://docs.n8n.io/integrations/builtin/credentials/google/)

---

## 📁 Repository Structure

```
n8n-workflows/
├── workflows/
│   └── Stock Market Monitor - Gemini AI + Gmail.json   # n8n workflow file
├── .github/
│   └── workflows/
│       └── validate.yml                                 # CI: JSON lint + secrets scan
├── .gitignore
└── README.md                                            # This file
```

---

## ⚙️ Configuration Reference

### Schedule

The workflow runs on a cron expression set inside the
`Daily Schedule (8AM Weekdays)` node:

```
0 8 * * 1-5
```

To change the time, open that node and edit the **Cron Expression** field.
For example, `0 7 * * 1-5` runs at 7:00 AM instead.

### Stock Universe

The full ticker lists are defined at the top of the
`Fetch All Stocks & Calculate` Code node:

```javascript
var usStocks = ['AAPL', 'MSFT', 'AMZN', ...]; // ~130 US tickers
var euStocks = ['SAP.DE', 'SIE.DE', ...];      // ~65 EU tickers
var ukStocks = ['SHEL.L', 'AZN.L', ...];       // ~63 UK tickers
```

Add or remove tickers by editing these arrays directly in the node.

### AI Prompt

The Gemini analysis prompt is configured in the `AI Investment Analysis`
node. The current prompt instructs the model to:

- Use `h3` tags for section headers
- Use `ul`/`li` for lists
- Apply `color: #d32f2f` for warnings and `color: #2e7d32` for opportunities
- Keep tone analytical, specific (use ticker symbols and numbers),
  and always include a "not financial advice" caveat

To adjust the analysis style or add/remove sections, edit the **Text**
field in that node.

### Gemini Model Settings

Configured inside the `Google Gemini Chat Model` node:

| Setting | Value |
|---|---|
| Model | `models/gemini-flash-latest` |
| Max Output Tokens | `16,096` |
| Temperature | `0.3` |

Lower temperature = more consistent, factual output.
Raise to `0.6–0.8` for more creative/narrative analysis.

### Rate Limiting

The workflow pauses **1.2 seconds every 5 tickers** to avoid hitting
Yahoo Finance rate limits:

```javascript
if ((ti + 1) % 5 === 0) await sleep(1200);
```

If you encounter 429 errors, increase this value (e.g., `2000` for 2s).

---

## 🩺 Troubleshooting

### Report not arriving in inbox

1. Check that the workflow is set to **Active** in n8n
2. Verify your Gmail OAuth2 credential is still valid
   (Google tokens expire — re-authenticate if needed)
3. Check n8n execution logs for errors on the `Send Gmail Report` node

### Ticker shows "Request failed with status code 404"

Yahoo Finance does not carry every ticker. Affected tickers are logged
in the **Data Retrieval Issues** table at the bottom of each report.
Remove problem tickers from the stock arrays in the Code node.

### AI analysis section is blank or shows a debug message

The `Combine AI Summary + Report` node reads the Gemini output from
`aiNode.output`. If your n8n version returns the response under a
different key, the node falls back through several alternatives.
Check your execution data to see the exact key Gemini returns and
update this line accordingly:

```javascript
if (aiNode.output) {
  aiAnalysis = aiNode.output;
}
```

### SMA/RSI calculation returns null

The workflow requires at least **180 days of closing price data**.
Newly listed stocks or tickers with limited history are skipped
and logged in the error table.

---

## 📀 How the Indicators Are Calculated

Both indicators are computed in the `Fetch All Stocks & Calculate`
Code node using raw closing prices from Yahoo Finance.

### 180-Day Simple Moving Average (SMA)

```javascript
function calcSMA(closes, period) {
  if (closes.length < period) return null;
  var slice = closes.slice(closes.length - period);
  var sum = 0;
  for (var i = 0; i < slice.length; i++) sum += slice[i];
  return sum / slice.length;
}
```

**Deviation %** = `((currentPrice - SMA) / SMA) × 100`

### RSI-14 (Relative Strength Index)

```javascript
function calcRSI(closes, period) {
  if (closes.length < period + 1) return null;
  var gains = 0, losses = 0;
  for (var i = closes.length - period; i < closes.length; i++) {
    var change = closes[i] - closes[i - 1];
    if (change > 0) gains += change;
    else losses += Math.abs(change);
  }
  var avgGain = gains / period;
  var avgLoss = losses / period;
  if (avgLoss === 0) return 100;
  return 100 - (100 / (1 + avgGain / avgLoss));
}
```

### Dividend Yield

Dividends are fetched by appending `&events=div` to the Yahoo Finance
API call. The workflow sums all dividend payments over the trailing
12 months and annualizes based on payment frequency.

---

## ⚠️ Disclaimer

> This workflow and the reports it generates are for **informational
> purposes only** and do **not** constitute financial advice. Stock market
> investing involves risk. Always conduct your own research or consult a
> qualified financial advisor before making any investment decisions.

---

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch: `git checkout -b feature/add-new-market`
3. Export your updated workflow from n8n as JSON
4. Replace the `.json` file and update this README if behaviour changes
5. Open a pull request with a clear description of what changed and why

---

## 📄 License

MIT License — Copyright (c) 2025 [Lucas Malik](https://github.com/EXL-1)

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute this workflow with attribution.
