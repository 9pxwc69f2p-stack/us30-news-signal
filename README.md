# us30-news-signal
fastapi
uvicorn
requests
pandas
numpy
spacy
scikit-learn
textblob
feedparser
apscheduler
web: uvicorn app.main:app --host 0.0.0.0 --port $PORT
{
  "build": {
    "builder": "NIXPACKS"
  }
}
from fastapi import FastAPI
from apscheduler.schedulers.background import BackgroundScheduler
from ingest import fetch_news
from processing import process_articles
from clustering import cluster_events
from prediction import predict_us30
from output import generate_signal
from datetime import datetime
import traceback

app = FastAPI(title="US30 News Signal API")

latest_signals = []
last_update_time = None
last_error = None

@app.get("/")
def root():
    return {"status": "running", "message": "US30 News Signal App Active"}

@app.get("/signals")
def get_signals():
    return latest_signals

@app.get("/diagnostics")
def diagnostics():
    return {
        "status": "ok" if last_error is None else "error",
        "last_update_time": last_update_time,
        "last_error": last_error,
        "signal_count": len(latest_signals),
        "topics": list({s.get("topic") for s in latest_signals}),
        "server_time": datetime.utcnow().isoformat()
    }

def update_signals():
    global latest_signals, last_update_time, last_error
    try:
        articles = fetch_news()
        processed = process_articles(articles)
        clusters = cluster_events(processed)

        signals = []
        for topic, cluster in clusters.items():
            prediction = predict_us30(cluster)
            signal = generate_signal(topic, prediction, cluster)
            signals.append(signal)

        latest_signals = signals
        last_update_time = datetime.utcnow().isoformat()
        last_error = None
    except Exception as e:
        last_error = traceback.format_exc()

scheduler = BackgroundScheduler()
scheduler.add_job(update_signals, "interval", minutes=5)
scheduler.start()

update_signals()
import feedparser

RSS_FEEDS = [
    "https://www.investing.com/rss/news_25.rss",
    "https://www.reuters.com/finance/markets/rss"
]

def fetch_news():
    articles = []
    for url in RSS_FEEDS:
        feed = feedparser.parse(url)
        for entry in feed.entries:
            articles.append({
                "title": entry.get("title", ""),
                "link": entry.get("link", ""),
                "summary": entry.get("summary", entry.get("title", "")),
            })
    return artist
    import spacy
from textblob import TextBlob

nlp = spacy.load("en_core_web_sm")

def clean_text(text):
    return text.replace("\n", " ").strip()

def extract_entities(text):
    return [ent.text for ent in nlp(text).ents]

def sentiment_analysis(text):
    polarity = TextBlob(text).sentiment.polarity
    if polarity > 0.05: return "positive"
    if polarity < -0.05: return "negative"
    return "neutral"

def classify_topic(text):
    keys = {
        "inflation": "US Economy",
        "jobs": "US Economy",
        "fed": "Federal Reserve",
        "rate": "Federal Reserve",
        "stocks": "Markets",
        "dow": "Markets",
        "geopolitics": "Geopolitics",
    }
    t = text.lower()
    for k, v in keys.items():
        if k in t:
            return v
    return "Other"

def process_articles(articles):
    processed = []
    for a in articles:
        text = clean_text(a["summary"])
        processed.append({
            "title": a["title"],
            "link": a["link"],
            "text": text,
            "entities": extract_entities(text),
            "sentiment": sentiment_analysis(text),
            "topic": classify_topic(text)
        })
    return processed
    def cluster_events(processed):
    clusters = {}
    for item in processed:
        clusters.setdefault(item["topic"], []).append(item)
    return clusters
    def predict_us30(cluster):
    pos = sum(1 for a in cluster if a["sentiment"] == "positive")
    neg = sum(1 for a in cluster if a["sentiment"] == "negative")

    if pos > neg:
        return {"direction": "LONG", "probability": pos / len(cluster)}
    elif neg > pos:
        return {"direction": "SHORT", "probability": neg / len(cluster)}
    else:
        return {"direction": "NEUTRAL", "probability": 0.5}
    from datetime import datetime
import uuid

def generate_signal(topic, prediction, cluster):
    return {
        "signal_id": str(uuid.uuid4()),
        "timestamp": datetime.utcnow().isoformat(),
        "topic": topic,
        "direction": prediction["direction"],
        "probability": prediction["probability"],
        "entities": list({e for a in cluster for e in a["entities"]}),
        "sources": [a["link"] for a in cluster]
    }
