
import os
import requests
import time
import csv
import json
from openai import OpenAI

# ----------------------------
# SSL Fix for Windows
# ----------------------------
os.environ.pop("SSL_CERT_FILE", None)

# ----------------------------
# CONFIG: Add your API keys
# ----------------------------
#Please dont steal my api keys  :D !!!
BEARER_TOKEN = "AND YOUR OWN"
OPENAI_API_KEY = "ADD YOUR OWN"

# ----------------------------
# Ask user for output folder
# ----------------------------
output_dir = input("📁 Enter the folder path where you'd like to save the results: ").strip()

if not os.path.exists(output_dir):
    os.makedirs(output_dir)

CSV_FILENAME = os.path.join(output_dir, "solana_tweets_with_evaluation.csv")
JSON_FILENAME = os.path.join(output_dir, "solana_tweets_with_evaluation.json")

# Initialize OpenAI client
client = OpenAI(api_key=OPENAI_API_KEY)

# ----------------------------
# SCORING CRITERIA DOCS (used in CSV/JSON)
# ----------------------------
SCORING_LOGIC = {
    "relevance": "Based on how viral or important the tweet seems. If it's the most retweeted, it's High; otherwise judged via tone or engagement level.",
    "risk": "High if tweet includes phrases like 'solana is done again', or is hypey, fear-based, or scammy. Low if informative or neutral.",
    "reliability": "Based on author's follower count: High = >100K, Medium = >10K, Low = <10K."
}

# ----------------------------
# Twitter API Functions
# ----------------------------
def create_headers(token):
    return {"Authorization": f"Bearer {token}"}

def fetch_recent_solana_tweets():
    url = "https://api.twitter.com/2/tweets/search/recent"
    params = {
        "query": "Solana",
        "max_results": 10,
        "tweet.fields": "created_at,text,public_metrics,author_id",
        "expansions": "author_id",
        "user.fields": "public_metrics"
    }

    headers = create_headers(BEARER_TOKEN)
    response = requests.get(url, headers=headers, params=params)

    print("\n--- TWITTER RESPONSE STATUS ---")
    print("Status Code:", response.status_code)

    if response.status_code in [401, 429]:
        reason = "unauthorized" if response.status_code == 401 else "rate limited"
        print(f"⚠️ API access {reason}. Using dummy tweets instead...\n")
        return get_dummy_tweets()

    if response.status_code != 200:
        print("❌ Error:", response.text)
        return []

    data = response.json()
    tweets = data.get("data", [])
    users = {u["id"]: u for u in data.get("includes", {}).get("users", [])}

    for tweet in tweets:
        tweet["user"] = users.get(tweet["author_id"], {})

    return tweets

def get_dummy_tweets():
    return [
        {
            "text": "Solana flips Ethereum? 🤯",
            "created_at": "2025-03-28",
            "public_metrics": {"retweet_count": 52},
            "user": {"public_metrics": {"followers_count": 120000}}
        },
        {
            "text": "Solana is going to $300 — book it.",
            "created_at": "2025-03-27",
            "public_metrics": {"retweet_count": 91},
            "user": {"public_metrics": {"followers_count": 7500}}
        },
        {
            "text": "solana is down / offline again. I told you. 🔥",
            "created_at": "2025-03-26",
            "public_metrics": {"retweet_count": 29},
            "user": {"public_metrics": {"followers_count": 15000}}
        },
    ]

# ----------------------------
# GPT Evaluation
# ----------------------------
def evaluate_tweet_with_gpt(tweet_text, follower_count):
    prompt = f"""
You are analyzing crypto-related tweets.

Tweet:
\"{tweet_text}\"

Author follower count: {follower_count}

Evaluate the tweet and return a JSON with:

- relevance: High / Medium / Low (how important or viral it seems)
- risk: High / Medium / Low   (how risky it seams to list the token on an exchange)
- reliability: High / Medium / Low (based on follower count — High = 100K+, Medium = 10K+, Low = under 10K)

Only return valid JSON like this:
{{
  "relevance": "...",
  "risk": "...",
  "reliability": "..."
}}
"""
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.5,
        max_tokens=150
    )
    return response.choices[0].message.content.strip()

# ----------------------------
# Local Evaluation Fallback
# ----------------------------
def local_tweet_evaluation(tweet, max_retweets):
    text = tweet["text"].lower()
    retweets = tweet["public_metrics"]["retweet_count"]
    followers = tweet.get("user", {}).get("public_metrics", {}).get("followers_count", 0)

    if retweets == max_retweets:
        relevance = "High"
    elif retweets >= max_retweets * 0.5:
        relevance = "Medium"
    else:
        relevance = "Low"

    risk = "High" if "solana is done again" in text else "Low"

    if followers >= 100_000:
        reliability = "High"
    elif followers >= 10_000:
        reliability = "Medium"
    else:
        reliability = "Low"

    return {
        "relevance": relevance,
        "risk": risk,
        "reliability": reliability,
        "source": "GPT failed: quota exceeded or no token. Fallback to local evaluation."
    }

# ----------------------------
# Save to CSV and JSON
# ----------------------------
def save_results(data):
    print(f"\n📁 Saving results to {CSV_FILENAME} and {JSON_FILENAME}...")

    # Save CSV with aligned logic explanations
    with open(CSV_FILENAME, mode="w", encoding="utf-8-sig", newline="") as file:
        writer = csv.writer(file, delimiter=';', quoting=csv.QUOTE_ALL)

        # Write custom explanation block
        writer.writerow(["# This file was generated from a limited X (Twitter) developer API."])
        writer.writerow(["# It does not scrape tweets from the full past 7 days,"])
        writer.writerow(["# because the X API only allows 1 request every 15 minutes and up to 100 posts total."])
        writer.writerow(["# GPT evaluation also requires tokens."])
        writer.writerow(["# Since the developer API key has no tokens left, a fallback logic was used instead."])
        writer.writerow(["# This code is easily scalable, but would require a paid X API and GPT tokens."])
        writer.writerow([])

        # Write explanation rows (like a report)
        writer.writerow(["# Evaluation Logic if GPT  has no token left:", "if gpt :"])
        writer.writerow([
            "# RELEVANCE: Based on how viral or important the tweet seems. If it's the most retweeted, it's High",
            "(how important or viral it seems)"
        ])
        writer.writerow([
            "# RISK: High if tweet includes phrases like 'solana is down/ offline again',",
            "risk: High / Medium / Low   (how risky it seams to list the token on an exchange)"
        ])
        writer.writerow([
            "# RELIABILITY: Based on author's follower count: High = >100K, Medium = >10K, Low = <10K.",
            "# RELIABILITY: Based on author's follower count: High = >100K, Medium = >10K, Low = <10K."
        ])
        writer.writerow([])  # Empty row for spacing

        # Data headers
        writer.writerow([
            "text", "created_at", "retweet_count", "follower_count",
            "relevance", "risk", "reliability", "evaluation_source"
        ])

        # Write tweet rows
        for row in data:
            writer.writerow([
                row["text"],
                row["created_at"],
                row["retweet_count"],
                row["follower_count"],
                row["relevance"],
                row["risk"],
                row["reliability"],
                row["evaluation_source"]
            ])

    # Save JSON with full criteria
    output = {
        "criteria_used": SCORING_LOGIC,
        "evaluated_tweets": data
    }

    with open(JSON_FILENAME, "w", encoding="utf-8") as f:
        json.dump(output, f, indent=2, ensure_ascii=False)

    print("✅ Files saved!")


# ----------------------------
# Main Execution
# ----------------------------
if __name__ == "__main__":
    tweets = fetch_recent_solana_tweets()
    if not tweets:
        print("No tweets found.")
        exit()

    print("\n🤖 Evaluating tweets using GPT or fallback logic:\n")
    max_retweets = max(tweet["public_metrics"]["retweet_count"] for tweet in tweets)
    evaluated_data = []

    for i, tweet in enumerate(tweets, 1):
        text = tweet["text"]
        created = tweet["created_at"]
        retweets = tweet["public_metrics"]["retweet_count"]
        followers = tweet.get("user", {}).get("public_metrics", {}).get("followers_count", 0)

        print(f"Tweet #{i}\nText: {text}\nRetweets: {retweets}, Followers: {followers}")

        try:
            gpt_json_str = evaluate_tweet_with_gpt(text, followers)
            rating = json.loads(gpt_json_str)
            rating["source"] = "gpt"
        except Exception as e:
            print("❌ GPT Error:", e)
            rating = local_tweet_evaluation(tweet, max_retweets)

        print("✅ Evaluation:", rating, "\n" + "-"*70)

        evaluated_data.append({
            "text": text,
            "created_at": created,
            "retweet_count": retweets,
            "follower_count": followers,
            "relevance": rating["relevance"],
            "risk": rating["risk"],
            "reliability": rating["reliability"],
            "evaluation_source": rating["source"]
        })

        time.sleep(1)  # Avoid GPT rate limit

    # Save to files
    save_results(evaluated_data)
