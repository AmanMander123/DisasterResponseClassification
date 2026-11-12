# Disaster message classification

This 2021 project explores how a responder could review incoming messages by category. A message can carry several labels, such as a request for water and a report of damaged infrastructure. The Flask interface shows predicted categories for a reader to review.

## From raw messages to the interface

1. [Data preparation](data/process_data.py) joins messages and labels by ID, expands the encoded categories into separate columns, and writes the cleaned records to SQLite.
2. [Model training](models/train_classifier.py) tokenizes the text, builds count and TF-IDF features, and tunes a multi-output random forest.
3. [The Flask app](app/run.py) displays dataset summaries and shows the predicted labels for a submitted message.

The historical evaluation holds out 20% of the records and reports precision, recall, F1, and support per label. Looking at labels individually matters because rare requests can be missed even when aggregate performance looks acceptable. The project is a workflow demonstration for human review.

## Rebuild the SQLite dataset

With pandas and SQLAlchemy installed in a local Python environment, run this from the repository root. The output path keeps the checked-in SQLite file intact.

```sh
demo_dir="$(mktemp -d)"
python data/process_data.py data/disaster_messages.csv data/disaster_categories.csv "$demo_dir/DisasterResponse.db"
python - "$demo_dir/DisasterResponse.db" <<'PY'
import sqlite3
import sys

with sqlite3.connect(sys.argv[1]) as connection:
    rows = connection.execute("SELECT COUNT(*) FROM DisasterResponse").fetchone()[0]
    columns = connection.execute("PRAGMA table_info(DisasterResponse)").fetchall()
print(f"{rows:,} messages and {len(columns) - 4} label columns")
PY
```

The local rebuild produced **26,028 messages and 36 label columns**. This verifies data preparation. Training and the Flask prediction path remain historical.

## Historical training and app

The [original environment export](requirements.txt) targets Intel macOS and Python 3.8. With the original environment and NLTK data available, run `python train_classifier.py ../data/DisasterResponse.db classifier.pkl` from `models/`, then run `python run.py` from `app/`. The Flask app expects the fitted model and SQLite file at those relative paths.
