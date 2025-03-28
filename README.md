# Automated-Asset-Monitoring---Coding-Challenge
Extraction of recent post about solana from X and analyze the relvance using an LLM


# 🔍 Solana Tweet Scraper & Evaluator (X API + GPT)

This Python tool fetches recent **Solana-related tweets** using the official **X (Twitter) API** and evaluates each tweet using **OpenAI GPT**.

If GPT tokens are unavailable or exhausted, it falls back to a **local evaluation logic**.  
All results are saved in an **Excel-friendly CSV** and a structured **JSON** format.

---

## ✨ Features

- 🐦 Fetches up to **100 recent tweets** about "Solana" using Twitter API v2
- 🧠 Evaluates each tweet for:
  - `relevance`: How viral/important the tweet is
  - `risk`: How risky it would be to list this token
  - `reliability`: Based on author's follower count
- 🤖 Uses **OpenAI GPT** (e.g., `gpt-3.5-turbo`) for intelligent scoring
- 🔁 Falls back to **rule-based logic** if GPT fails
- 📁 Saves results to:
  - `CSV` (Excel-friendly, with semicolon delimiter + UTF-8 BOM)
  - `JSON` (with scoring metadata)
- 📋 Adds clear comments and logic explanation in the CSV header

---

## ⚙️ Requirements

- Python 3.7+
- An X (Twitter) **developer API key**
- An OpenAI **API key**

Install dependencies:

```bash
pip install openai requests
```

---

## 🔐 API Keys

You need to set two keys in the script:

```python
BEARER_TOKEN = "your-twitter-bearer-token"
OPENAI_API_KEY = "your-openai-api-key"
```

---

## 🚀 How to Run

```bash
python solana_scraper.py
```

The script will prompt you to choose a folder for saving results. Please copy your path of the folder.

---

## 🧠 Scoring Criteria

> Used by GPT and fallback logic if GPT fails.

| Category       | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| **Relevance**  | High = Most retweeted tweet; Medium/Low based on tone or engagement         |
| **Risk**       | High if tweet includes fear, hype, or phrases like “solana is down/offline” |
| **Reliability**| Based on author follower count: High = 100K+, Medium = 10K+, Low < 10K       |

---

## 📦 Output Files

### 🗂 CSV

- Saved with semicolon delimiter (`;`) and `UTF-8-BOM` encoding for Excel compatibility
- Includes explanatory comment block at the top

```
# This file was generated from a limited X (Twitter) developer API.
# It does not scrape tweets from the full past 7 days,
# because the X API only allows 1 request every 15 minutes and up to 100 posts total.
# GPT evaluation also requires tokens.
# Since the developer API key has no tokens left, a fallback logic was used instead.
# This code is easily scalable, but would require a paid X API and GPT tokens.

# Evaluation Logic if GPT has no token left: ; if gpt :
# RELEVANCE: Based on how many retweets
# RISK: High if tweet includes phrases like 'solana is down/ offline again', ; risk: High / Medium / Low    (sorry for the solana joke)
# RELIABILITY: Based on author's follower count: High = >100K, Medium = >10K, Low = <10K. ; Same for GPT
```

### 🧾 JSON

Includes structured output:

```json
{
  "criteria_used": {
    "relevance": "...",
    "risk": "...",
    "reliability": "..."
  },
  "evaluated_tweets": [
    {
      "text": "...",
      "relevance": "High",
      "risk": "Low",
      "reliability": "Medium",
      ...
    }
  ]
}
```

---

## ⚠️ Limitations (Free API Mode)

- Twitter API allows only **1 request every 15 minutes**
- Max of **100 tweets** per request
- GPT requires **paid API tokens** — fallback logic used when tokens are unavailable

---

## 🧪 Sample Fallback Evaluation

If GPT is unavailable:

```text
Tweet: "solana is down / offline again. I told you. 🔥"
Followers: 15,000
→ Relevance: Low
→ Risk: High
→ Reliability: Medium
→ Source: GPT failed, fallback logic used
```

---

## 📈 Scalability Ideas

- Add cron jobs or Airflow for scheduled scraping
- Add charts, alerts or dashboards using the JSON
- Upgrade to full X API access and GPT token pool

---

## 📄 License

MIT — free to use and modify.

---

## 👨‍💻 Author

Built by ME 
Powered by 🧠 OpenAI + 🐦 Twitter API
