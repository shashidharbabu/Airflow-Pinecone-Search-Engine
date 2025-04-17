# Medium-to‑Pinecone Search Engine

A simple Airflow-based pipeline to build a semantic search engine over a Medium article dataset, using `sentence-transformers` for text embeddings and Pinecone as a vector store. This repo includes:

- A custom Airflow DAG (`build_pinecone_search.py`)
- A `docker-compose.yaml` configured to install dependencies
- Instructions to configure Pinecone credentials, run the pipeline, and verify each step
- A sample preprocessed CSV ready for ingestion

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Repository Structure](#repository-structure)
3. [Setup & Installation](#setup--installation)
4. [Configure Pinecone API Key](#configure-pinecone-api-key)
5. [Run the Airflow Pipeline](#run-the-airflow-pipeline)
6. [Export Preprocessed Data](#export-preprocessed-data)
7. [Verify Search](#verify-search)
8. [Screenshots & Reporting](#screenshots--reporting)
9. [GitHub Submission](#github-submission)

---

## Prerequisites

- Docker >= 20.10 & Docker Compose V2
- A Pinecone account (free tier works)
- Internet access to download the Medium dataset

---

## Repository Structure

```
.
├── build_pinecone_search.py   # Airflow DAG definition
├── docker-compose.yaml        # Airflow + Postgres services, with custom pip deps
├── dags/                      # (symlinked) contains the DAG file
├── logs/                      # Airflow logs
└── README.md                  # This file
```

---

## Setup & Installation

1. **Clone the repo**
   ```bash
   git clone https://github.com/<your‑username>/medium-to-pinecone.git
   cd medium-to-pinecone
   ```
2. **Edit** `docker-compose.yaml`
   Under `x-airflow-common.environment`, ensure:
   ```yaml
   _PIP_ADDITIONAL_REQUIREMENTS: ${_PIP_ADDITIONAL_REQUIREMENTS:- sentence-transformers pinecone-client yfinance apache-airflow-providers-snowflake snowflake-connector-python}
   ```
3. **Bring up containers**
   ```bash
   docker compose down
   docker compose up -d
   ```
4. **Verify** in the `airflow` service logs that `sentence-transformers` & `pinecone-client` installed successfully.

---

## Configure Pinecone API Key

1. Obtain your API key from [Pinecone Console](https://app.pinecone.io) → **API Keys**.
2. In Airflow’s UI (http://localhost:8081) go to **Admin → Variables** → **+** and add:
   - **Key**: `pinecone_api_key`
   - **Value**: `<YOUR_PINECONE_API_KEY>`
3. Confirm it appears in the list.

_Alternative via CLI:_
```bash
docker compose exec airflow-cli \
  airflow variables set pinecone_api_key <YOUR_PINECONE_API_KEY>
```

---

## Run the Airflow Pipeline

1. In the Airflow UI, **unpause** the `Medium_to_Pinecone` DAG.
2. Click ▶️ **Trigger DAG**.
3. Monitor the graph:

   - **download_data**: downloads `medium_data.csv` (~X lines)
   - **preprocess_data**: writes `/tmp/medium_data/medium_preprocessed.csv`
   - **create_pinecone_index**: creates (or resets) index `semantic-search-fast`
   - **generate_embeddings_and_upsert**: batches embeddings via `all-MiniLM-L6-v2` and upserts
   - **test_search_query**: runs a sample query and prints top-5 matches

Capture screenshots of each task’s **View Log** once it succeeds.

---

## Export Preprocessed Data

To pull the preprocessed CSV out of the container:
```bash
docker compose exec airflow \
  bash -c "cat /tmp/medium_data/medium_preprocessed.csv" \
  > medium_preprocessed.csv
```
Verify locally:
```bash
ls -lh medium_preprocessed.csv
head medium_preprocessed.csv
```

---

## Verify Search

You can test queries interactively:

```python
from sentence_transformers import SentenceTransformer
from pinecone import Pinecone

api_key = "<YOUR_PINECONE_API_KEY>"
model   = SentenceTransformer('all-MiniLM-L6-v2')
pc      = Pinecone(api_key=api_key)
index   = pc.Index("semantic-search-fast")

query = "ethics in AI"
vec   = model.encode(query).tolist()
res   = index.query(vector=vec, top_k=5, include_metadata=True)

for m in res["matches"]:
    print(m["score"], m["metadata"]["title"])
```

---

## Screenshots & Reporting

Airflow DAGs
![Screenshot 2025-04-16 at 22 39 36](https://github.com/user-attachments/assets/4e50cb6e-94f7-4e36-88ac-30d1caf13a0f)

Pinecone Search Engine Results
![Screenshot 2025-04-16 at 22 52 15](https://github.com/user-attachments/assets/adff6d3b-8ad2-4555-8796-bdf04fef83a9)




---

Name: Shashidhar Babu P V D
SID: 018174494
