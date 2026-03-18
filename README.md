# Market Price Tracker Agent — Concept Note

## Overview
An AI agent that monitors and reports commodity prices, forex rates, and market data relevant to East African businesses. Users subscribe to price alerts via WhatsApp/Telegram and get real-time updates when prices move.

## Problem
Traders, farmers, and businesses in East Africa make buying/selling decisions based on outdated price information. Commodity prices (maize, beans, coffee, fuel) fluctuate daily across markets. Forex rates change hourly. Getting accurate, timely data requires checking multiple sources manually.

## Solution
A configurable price tracking agent that:
- Scrapes/aggregates prices from multiple sources (UNBS, commodity exchanges, forex APIs, fuel stations)
- Sends price alerts when thresholds are hit
- Provides daily/weekly market summaries
- Answers natural language queries ("What's the price of maize in Kampala today?")
- Tracks historical trends and provides simple analytics
- Supports custom watchlists

## Architecture
```
┌─────────────────────────────────────────────┐
│            Market Price Tracker Agent         │
├─────────────────────────────────────────────┤
│  Data Sources     │  Core Engine             │
│  ├─ Forex APIs    │  ├─ LLM Provider (any)   │
│  ├─ Commodity DBs │  ├─ Price Aggregator      │
│  ├─ Web Scrapers  │  ├─ Alert Engine          │
│  └─ RSS/News      │  ├─ Trend Analyzer        │
│                   │  └─ Report Generator      │
│  Channels         │                          │
│  ├─ WhatsApp      │  Storage                 │
│  ├─ Telegram      │  ├─ SQLite (prices)       │
│  └─ Web Dashboard │  └─ Time-series data      │
└─────────────────────────────────────────────┘
```

## Tech Stack
- **Runtime:** Node.js
- **LLM:** Any provider (for natural language queries and report generation)
- **Data:** Web scraping (Cheerio/Puppeteer), REST APIs, RSS feeds
- **Channels:** Baileys (WhatsApp), Telegraf (Telegram), Express (Web)
- **Database:** SQLite (price history, subscriptions, alerts)
- **Scheduler:** node-cron (regular price fetches)

## Configuration
```bash
git clone https://github.com/AiStudioUg/market-price-tracker.git
cd market-price-tracker
cp .env.example .env
nano config/tracker.yaml
npm install && npm start
```

### config/tracker.yaml
```yaml
region: "east-africa"  # east-africa | uganda | kenya | tanzania

data_sources:
  forex:
    enabled: true
    pairs: ["USD/UGX", "EUR/UGX", "GBP/UGX", "KES/UGX"]
    update_interval: "30m"
  
  commodities:
    enabled: true
    items:
      - name: "Maize"
        unit: "kg"
        markets: ["Kampala", "Jinja", "Gulu", "Mbarara"]
      - name: "Beans"
        unit: "kg"
        markets: ["Kampala", "Mbale"]
      - name: "Coffee (Robusta)"
        unit: "kg"
        markets: ["Kampala"]
      - name: "Rice"
        unit: "kg"
        markets: ["Kampala", "Jinja"]
    update_interval: "6h"
  
  fuel:
    enabled: true
    types: ["petrol", "diesel", "kerosene"]
    update_interval: "24h"

alerts:
  default_threshold: 5  # percentage change to trigger alert

reports:
  daily_summary: true
  daily_time: "07:00"  # EAT
  weekly_summary: true
  weekly_day: "monday"

llm:
  provider: "openai"
  model: "gpt-4o-mini"
  api_key: "${OPENAI_API_KEY}"
```

## Conversation Example
```
User: What's maize going for in Kampala?
Agent: 📊 Maize Prices — Kampala (March 18, 2026)
       
       Current: UGX 1,800/kg
       Yesterday: UGX 1,750/kg (+2.9%)
       Last week: UGX 1,650/kg (+9.1%)
       
       📈 Trending UP — prices have risen 9% this week
       likely due to seasonal supply constraints.
       
       Other markets:
       • Jinja: UGX 1,700/kg
       • Gulu: UGX 1,500/kg
       • Mbarara: UGX 1,600/kg
       
       Want me to alert you when maize drops below UGX 1,700?

User: Yes, and also track USD/UGX
Agent: ✅ Set up:
       • Alert: Maize drops below UGX 1,700/kg in Kampala
       • Daily: USD/UGX rate at 7:00 AM
       
       Current USD/UGX: 3,785 (Bank of Uganda mid-rate)
```

## Browser Extension
- Dashboard showing watchlist prices at a glance
- Quick subscribe to commodities/forex via popup
- Price charts and trends
- Export data to CSV

## File Structure
```
market-price-tracker/
├── README.md
├── LICENSE
├── .env.example
├── package.json
├── config/
│   ├── tracker.yaml.example
│   └── sources/
│       ├── forex.yaml
│       ├── commodities.yaml
│       └── fuel.yaml
├── src/
│   ├── index.js
│   ├── agent/
│   │   ├── core.js
│   │   ├── query.js          # Natural language queries
│   │   └── reports.js        # Summary generation
│   ├── data/
│   │   ├── scraper.js        # Web scraping
│   │   ├── forex-api.js      # Forex data
│   │   ├── aggregator.js     # Price aggregation
│   │   └── scheduler.js      # Cron-based fetching
│   ├── alerts/
│   │   ├── engine.js         # Alert matching
│   │   └── notifier.js       # Send alerts
│   ├── channels/
│   │   ├── whatsapp.js
│   │   ├── telegram.js
│   │   └── web.js
│   ├── db/
│   │   └── sqlite.js
│   └── utils/
├── extension/
├── docs/
│   ├── setup.md
│   ├── data-sources.md
│   ├── alerts.md
│   └── adding-sources.md
└── tests/
```

## Success Metrics
- Price data accuracy > 95% vs reference sources
- Alert delivery within 5 minutes of threshold breach
- Daily active users tracking at least 3 items

## Target Users
- Agricultural traders and farmers
- Forex bureaus and money changers
- Import/export businesses
- Fuel station operators
- Financial analysts
- Anyone who buys/sells commodities
