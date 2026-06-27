# 📊 Crypto Portfolio Analytics & Alert System

> An automated no-code workflow built with **n8n** that tracks a cryptocurrency portfolio in real-time, calculates weighted performance metrics, and delivers instant **Telegram alerts** — running on a daily schedule with zero manual intervention.

---

## 🚀 Live Demo

| Status | Trigger | Telegram Output |
|---|---|---|
| 🔴 TRIGGERED | Weighted 24h change > ±0.1% | Volatility alert with full breakdown |
| 🟢 NORMAL | Weighted 24h change within ±0.1% | Daily summary report |

---
## 📸 Workflow Screenshot

![n8n Workflow](https://raw.githubusercontent.com/Apoo3va/crypto-portfolio-n8n/main/workflow/workflow-screenshot.png)

## 🧠 What It Does

This workflow automatically:

1. **Fetches live prices** for Bitcoin and Ethereum from the CoinGecko public API every day at 9:00 AM
2. **Calculates portfolio metrics** using weighted average analysis across all holdings
3. **Evaluates volatility** and flags a status of `TRIGGERED` or `NORMAL`
4. **Sends a Telegram message** with a full breakdown — every single day, regardless of market condition

---

## 🏗️ Workflow Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌────────────┐     ┌────────┐
│ Schedule Trigger │────▶│ HTTP Request │────▶│    Code    │────▶│   If   │
│  Daily @ 9 AM   │     │  CoinGecko   │     │ JS Engine  │     │  Node  │
└─────────────────┘     └──────────────┘     └────────────┘     └────────┘
                                                                     │
                                              ┌──────────────────────┤
                                              │ TRUE                 │ FALSE
                                              ▼                      ▼
                                    ┌──────────────────┐  ┌──────────────────┐
                                    │   Edit Fields    │  │  Edit Fields     │
                                    │ 🚨 Alert Report  │  │ ✅ Normal Report  │
                                    └──────────────────┘  └──────────────────┘
                                              │                      │
                                              ▼                      ▼
                                    ┌──────────────────┐  ┌──────────────────┐
                                    │ Telegram Alert   │  │ Telegram Normal  │
                                    │   (Send Msg)     │  │   (Send Msg)     │
                                    └──────────────────┘  └──────────────────┘
```

---

## ⚙️ Node Breakdown

### 1. Schedule Trigger
- Fires daily at **9:00 AM** using cron expression `0 9 * * *`
- Fully configurable to any interval

### 2. HTTP Request
- Calls the **CoinGecko Free API** (no API key required)
- Endpoint: `https://api.coingecko.com/api/v3/simple/price`
- Fetches: `price (USD)`, `24h volume`, `24h % change` for BTC & ETH

### 3. Code Node (JavaScript)
Core analytics engine. Calculates:

```js
// Portfolio holdings (configurable)
const btcHolding = 0.5;   // 0.5 BTC
const ethHolding = 3;     // 3 ETH

// Weighted 24h change formula
const weightedChange = (btcWeight * btcChange) + (ethWeight * ethChange);

// Alert logic
const alertStatus = (weightedChange > 0.1 || weightedChange < -0.1)
  ? 'TRIGGERED'
  : 'NORMAL';
```

**Output fields:**
| Field | Description |
|---|---|
| `btcPrice` / `ethPrice` | Current USD price |
| `btcValue` / `ethValue` | USD value of holdings |
| `totalPortfolioValue` | Combined portfolio value |
| `weightedDailyChange` | Portfolio-weighted 24h % change |
| `alertStatus` | `TRIGGERED` or `NORMAL` |

### 4. If Node
- Evaluates `alertStatus === "TRIGGERED"`
- Routes to the appropriate branch

### 5. Edit Fields + Telegram (×2)
- **TRUE branch**: Formats and sends a 🚨 volatility alert
- **FALSE branch**: Formats and sends a ✅ daily normal summary

---

## 📦 Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** (v1.0+) | Workflow automation engine |
| **CoinGecko API** | Free real-time crypto price data |
| **Telegram Bot API** | Push notification delivery |
| **JavaScript** | Portfolio analytics & calculations |

---

## 🛠️ Setup & Installation

### Prerequisites
- n8n instance (self-hosted or cloud)
- Telegram account

### Step 1 — Import Workflow
1. Open n8n canvas
2. Click `...` menu → **Import from JSON**
3. Upload `workflow/crypto_portfolio_workflow.json`

### Step 2 — Create Telegram Bot
1. Open Telegram → search **@BotFather**
2. Send `/newbot` and follow prompts
3. Copy the **Bot Token**

### Step 3 — Configure n8n Credentials
1. Click the Telegram node → **Credentials** → **Create New**
2. Paste your Bot Token → Save

### Step 4 — Get Your Chat ID
1. Send `/start` to your bot in Telegram
2. Visit: `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
3. Copy the `chat.id` value from the response

### Step 5 — Set Chat ID
Update the `chatId` field in both Telegram nodes with your Chat ID number.

### Step 6 — Activate
Toggle the **Active** switch in n8n → workflow runs daily at 9 AM automatically.

---

## 🔧 Configuration

To customize the portfolio, edit the **Code node**:

```js
// Change these values to match your actual holdings
const btcHolding = 0.5;   // How many BTC you own
const ethHolding = 3;     // How many ETH you own

// Change alert sensitivity (default ±0.1%)
const alertStatus = (weightedChange > 0.1 || weightedChange < -0.1)
  ? 'TRIGGERED' : 'NORMAL';
```

---

## 📱 Sample Telegram Output

**Alert Message:**
```
🚨 CRYPTO PORTFOLIO ALERT — VOLATILITY TRIGGERED
📅 2025-01-15T09:00:00.000Z

📊 Portfolio Summary
💰 Total Value: $42,850.00 USD
📉 Weighted 24h Change: -2.3400%
🔴 Alert Status: TRIGGERED

🪙 Bitcoin (BTC)
• Holdings: 0.5 BTC
• Price: $65,200.00
• Value: $32,600.00
• 24h Change: -2.1%

🪙 Ethereum (ETH)
• Holdings: 3 ETH
• Price: $3,416.67
• Value: $10,250.00
• 24h Change: -3.1%

⚠️ High Volatility Warning
Your portfolio moved more than ±0.1% in the last 24h.
```

---

## 🗂️ Project Structure

```
crypto-portfolio-n8n/
├── README.md                          # This file
├── workflow/
│   └── crypto_portfolio_workflow.json # n8n workflow (import this)
└── docs/
    └── SETUP.md                       # Detailed setup guide
```

---

## 🔮 Future Improvements

- [ ] Add support for more assets (SOL, BNB, XRP)
- [ ] Google Sheets logging for historical tracking
- [ ] Weekly/monthly performance summary reports
- [ ] Email delivery via Gmail node
- [ ] Portfolio rebalancing suggestions

---

## 👤 Author - Apoorva Yadav
Built as a portfolio project demonstrating **workflow automation**, **API integration**, **real-time analytics**, and **event-driven notification systems** using n8n.

---

## 📄 License

MIT License — free to use, modify, and distribute.
