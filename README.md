# Real-Time Twitter Sentiment & Entity Analysis (Airport Passenger Feedback)

An interactive Jupyter notebook that pulls live tweets for a search keyword, scores their sentiment, and breaks down feedback by topic — built around airport passenger-experience monitoring (the sample search context is `CSMIA_official`, Mumbai's Chhatrapati Shivaji Maharaj International Airport).

Type a keyword into a search box, hit enter, and the notebook fetches recent tweets, cleans the text, scores sentiment, classifies it with a trained model, extracts named entities, and renders word clouds and trend charts — all inline.

## What it does

1. **Search UI** — an `ipywidgets` text box (`inputText`) triggers the full pipeline on submit. Built to also run standalone as a dashboard via [Voila](https://voila.readthedocs.io/).
2. **Tweet collection** (`collect_tweets`) — fetches the most recent 250 tweets matching the search term (excluding retweets) via the Tweepy v1.1 API, and pulls 7-day hourly/daily tweet-volume counts via Twarc2 (API v2).
3. **Preprocessing** (`clean`, `deEmojify`) — strips handles, links, hashtags, punctuation, and emoji; lemmatizes to root words; drops non-English tokens.
4. **Sentiment scoring** (`ps_scores`) — computes Polarity and Subjectivity per tweet with TextBlob, then classifies each tweet into a sentiment category using a pre-trained scikit-learn `LogisticRegression` model (`finalized_model.sav`, 3 classes).
5. **Entity tagging** — two parallel approaches:
   - **Dynamic entities**: spaCy NER (`dynamic_entity_table`) extracts named entities directly from tweet text, with fuzzy de-duplication (`fuzzywuzzy`) to merge near-identical entity names.
   - **Predefined entities** (`pre-defined_entities.xlsx`): a curated keyword → category map (e.g. *toilet/washroom → Hygiene Related*, *staff → Staff Related*, *lounge, security, checkin, luggage, food, parking, rtpcr/quarantine → Covid*) used to tally sentiment counts per topic.
6. **Visualization** — word clouds (overall / positive / negative), daily & hourly tweet-volume charts, a 7-day stacked sentiment trend chart, and an overall sentiment trend line (via Plotly/Cufflinks + Matplotlib/Seaborn).
7. **Summaries** — top 5 most-retweeted, most-liked, and most-negative-but-retweeted tweets printed for quick scanning.
8. **Historical rollup** — new results are appended to a running consolidated dataset (`consolidated_till2021-11-30.xlsx`) so trend charts reflect history, not just the current search.

## Repo contents

| File | Description |
|---|---|
| `20211216_Realtime_Tweets_Sentiment_Voila_Interactive_v1.ipynb` | Main notebook — all pipeline functions + the interactive search widget |
| `pre-defined_entities.xlsx` | Keyword → category lookup table used for topic tagging |
| `sample_raw_tweets.csv` | **Synthetic** example data (fabricated, no real tweets/PII) showing the raw tweet schema collected by `collect_tweets`/`tweets_dataframe` |
| `sample_processed_output.csv` | **Synthetic** example showing what the pipeline outputs after cleaning, sentiment scoring, and entity tagging |
| `.env.example` | Template for required API credentials |
| `requirements.txt` | Python dependencies |

**Not included** (kept local — not needed to understand the code, and contain real data/artifacts): the trained model file (`finalized_model.sav`), the historical consolidated dataset, and the raw `AllTweets_*.xlsx` export snapshots, since these contain real passenger feedback. The supporting slide deck (`Twitter Data Analysis.pptx`) walking through the methodology also remains local.

## Setup

```bash
pip install -r requirements.txt
python -m spacy download en_core_web_sm
python -m nltk.downloader stopwords words wordnet vader_lexicon
```

Copy `.env.example` to `.env` and fill in your own Twitter API credentials (a [developer.twitter.com](https://developer.twitter.com) account is required — v1.1 keys for `tweepy`, plus a v2 bearer token for `twarc2`):

```bash
cp .env.example .env
```

Run the notebook normally with Jupyter, or as a standalone interactive app with Voila:

```bash
voila 20211216_Realtime_Tweets_Sentiment_Voila_Interactive_v1.ipynb
```

## Security note

The original notebook had live API credentials hardcoded in the search-widget cell. This version loads them from environment variables (`os.environ[...]`) via `python-dotenv` instead — `.env` is git-ignored, so secrets never get committed.

## Background

Notebook dated December 2021; the consolidated dataset spans tweets/feedback through late 2021–early 2023. Originally built as a passenger-experience monitoring tool — surfacing recurring complaint themes (hygiene, staff conduct, security wait times, COVID procedures, etc.) from social media in near real time.
