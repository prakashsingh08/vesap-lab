---
# vespa-lab

A self-contained local learning environment for [Vespa](https://vespa.ai) — the open-source platform for high-scale search, recommendation, and ranking. The lab runs entirely via Docker Compose and progresses through 12 structured modules covering every core Vespa concept from basic full-text search to hybrid vector-keyword retrieval.

---
## What this lab covers

| Module | Topic |
|--------|-------|
| 00 | Search Fundamentals — inverted index, tokenization, recall/precision |
| 01 | Vespa Architecture — config cluster, container cluster, content cluster, Proton |
| 02 | Document Model — field types, structs, arrays, weightedsets, partial updates |
| 03 | Indexing — text processing pipeline, match modes, memindex, schema evolution |
| 04 | Search & YQL — query language, operators, pagination, annotations |
| 05 | BM25 — TF-IDF progression, k1/b parameters, score decomposition |
| 06 | Ranking — rank profiles, two-phase ranking, rank features, freshness decay |
| 07 | Attributes — column store, fast-search, sorting, filtering |
| 08 | Grouping — faceted aggregation, nested groups, numeric bucketing |
| 09 | Tensors — indexed/mapped types, dot product, cosine similarity |
| 10 | Vector Search — HNSW index, nearestNeighbor, distance metrics, targetHits |
| 11 | Hybrid Search — BM25 + ANN fusion, score normalization, dual retrieval |

---
## Repository layout

```
vespa_lab/
├── docker/
│   ├── docker-compose.yml
│   ├── .env
│   └── README.md
├── applications/
│   └── my-app/
│       ├── services.xml
│       └── schemas/
│           └── article.sd
├── feeds/
│   └── articles.jsonl
├── specs/
│   ├── module-00-search-fundamentals/
│   ├── module-01-vespa-architecture/
│   ├── module-02-document-model/
│   ├── module-03-indexing/
│   ├── module-04-search/
│   ├── module-05-bm25/
│   ├── module-06-ranking/
│   ├── module-07-attributes/
│   ├── module-08-grouping/
│   ├── module-09-tensors/
│   ├── module-10-vector-search/
│   └── module-11-hybrid-search/
│       (each module: spec.md · exercises.md · notes.md · tasks.md)
├── docs/
├── queries/
├── scripts/
└── journal/
```

---
## Prerequisites

- Docker Desktop (macOS/Windows) or Docker Engine (Linux)
- `curl`, `python3` (for exercise scripts)
- ~4 GB free RAM for the Vespa container

---
## Quick start

```bash
# 1. Start all containers
cd docker
docker compose --env-file .env up -d

# 2. Wait ~30 seconds for Vespa to initialise, then verify
curl -s http://localhost:8080/state/v1/health | python3 -m json.tool

# 3. Deploy the application package
cd ../applications/my-app
zip -r ../my-app.zip .
curl -X POST http://localhost:19071/application/v2/tenant/default/prepareandactivate \
  -H "Content-Type: application/zip" \
  --data-binary @../my-app.zip

# 4. Feed sample documents
curl -X POST http://localhost:8080/document/v1/default/article/docid/1 \
  -H "Content-Type: application/json" \
  --data-binary @../../feeds/articles.jsonl
```

Full deploy and feed instructions are in docker/README.md.

---
## Endpoints
```
┌───────────┬────────────────────────────────────┬────────────────────┐
│  Service  │                URL                 │      Purpose       │
├───────────┼────────────────────────────────────┼────────────────────┤
│ Query API │ http://localhost:8080/search/      │ Run YQL queries    │
├───────────┼────────────────────────────────────┼────────────────────┤
│ Document  │ http://localhost:8080/document/v1/ │ Feed / get /       │
│ API       │                                    │ delete documents   │
├───────────┼────────────────────────────────────┼────────────────────┤
│ Config    │ http://localhost:19071             │ Deploy application │
│ API       │                                    │  packages          │
├───────────┼────────────────────────────────────┼────────────────────┤
│ Vispana   │ http://localhost:4000              │ Browse schema and  │
│ UI        │                                    │ documents visually │
└───────────┴────────────────────────────────────┴────────────────────┘
```
---
## How to use the learning modules

Each module under specs/ contains four files:

1. spec.md — read this first; explains the concept with diagrams and examples
2. exercises.md — run every command against the live Vespa instance
3. notes.md — record your actual observed values in the observation slots
4. tasks.md — tick off completed items; answer gate questions from memory before advancing

Work through modules in order (00 → 11). Each module builds on the schema and concepts established by the previous one.

---
## Stack
```
┌─────────────┬──────────────────────────┬─────────────────────────────┐
│  Component  │          Image           │            Role             │
├─────────────┼──────────────────────────┼─────────────────────────────┤
│ Vespa       │ vespaengine/vespa:latest │ Search engine (config +     │
│             │                          │ container + content)        │
├─────────────┼──────────────────────────┼─────────────────────────────┤
│ Vispana     │ vispana/vispana          │ Web UI for schema and       │
│             │                          │ document inspection         │
├─────────────┼──────────────────────────┼─────────────────────────────┤
│ feed-client │ python:3.12-slim         │ Utility container with curl │
│             │                          │  for feeding                │
└─────────────┴──────────────────────────┴─────────────────────────────┘
```

