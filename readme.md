# Ejemplo practico del reto Dataton 2022

```mermaid
flowchart LR
  subgraph Ingest
    A1["APIs: Perplexity / OpenAI / X (hashtags, likes, growth)"] -->|stream / batch| K1["Kafka / PubSub"]
    A2["Walmart / OpenFoodFacts (ventas, precios)"] --> K1
    A3["Historial interno (recetas, performance)"] --> K1
    A4["Dataset nutricional público"] --> K1
  end

  K1 --> ETL["ETL / Data Cleaning (Airflow)"] --> FS["Feature Store / Time-series DB"]
  FS --> TrendEngine["Trend detection<br/>(growth models, anomaly detection,<br/>topic clustering, embeddings)"]

  TrendEngine --> CandidateGen["Candidate generator<br/>(RAG + retrieval, template + parametric gen)"]
  CandidateGen --> RankModel["Ranking & Business rules"]
  RankModel --> Product["API / Frontend + CMS"]

  Product --> Analytics["Event tracking (clicks, saves, CTR)"] --> K1
  RankModel --> OnlineStore["Redis cache / Feature serving"]
  TrendEngine --> Monitoring["Model & Data drift monitoring"]

```mermaid
flowchart LR
  subgraph Bronze["Bronze Layer (Raw)"]
    A1["APIs: Perplexity / OpenAI / X (hashtags, likes, growth)"] --> K1["Kafka / PubSub"]
    A2["Walmart / OpenFoodFacts (ventas, precios)"] --> K1
    A3["Historial interno (recetas, performance)"] --> K1
    A4["Dataset nutricional público"] --> K1
  end

  subgraph Silver["Silver Layer (Cleansed)"]
    K1 --> ETL["ETL / Data Cleaning (Airflow)"]
    ETL --> FS["Feature Store / Time-series DB"]
  end

  subgraph Gold["Gold Layer (Business-ready)"]
    FS --> TrendEngine["Trend detection<br/>(growth models, anomaly detection,<br/>topic clustering, embeddings)"]
    TrendEngine --> CandidateGen["Candidate generator<br/>(RAG + retrieval, template + parametric gen)"]
    CandidateGen --> RankModel["Ranking & Business rules"]
    RankModel --> Product["API / Frontend + CMS"]
    Product --> Analytics["Event tracking (clicks, saves, CTR)"]
  end

  %% Feedback loops
  Analytics --> K1
  RankModel --> OnlineStore["Redis cache / Feature serving"]
  TrendEngine --> Monitoring["Model & Data drift monitoring"]
