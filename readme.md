# Ejemplo practico del reto Dataton 2022


# freshFork – Sistema de Recomendación Proactivo

## 🔹 Arquitectura Propuesta (Azure + Databricks Medallion)

```mermaid
flowchart LR
  subgraph Bronze["Bronze Layer (Raw Ingest)"]
    A1["APIs: Perplexity / OpenAI / X"] --> ADLS["Azure Data Lake Storage Gen2 (Raw)"]
    A2["Walmart / OpenFoodFacts"] --> ADLS
    A3["Historial interno (recetas, performance)"] --> ADLS
    A4["Dataset nutricional público"] --> ADLS
  end

  ADLS --> DLT["Azure Databricks (Delta Live Tables) - ETL / Quality"]

  subgraph Silver["Silver Layer (Cleansed / Enriched)"]
    DLT --> FS["Databricks Feature Store"]
    DLT --> TSDB["Delta Lake (time-series enriched data)"]
  end

  subgraph Gold["Gold Layer (Business-Ready)"]
    FS --> TrendEngine["Trend Detection Models<br/>(Prophet, ARIMA, Embeddings, Clustering)"]
    TSDB --> TrendEngine

    TrendEngine --> CandidateGen["Candidate Generator<br/>(RAG con Cognitive Search + Azure OpenAI)"]
    CandidateGen --> RankModel["Ranking (XGBoost / LightGBM)<br/>+ Business Rules"]

    RankModel --> Serving["Azure Databricks Model Serving<br/>+ Redis Cache (low-latency)"]
    Serving --> API["API Management + CMS Frontend"]
  end

  API --> Analytics["Azure Application Insights + Power BI<br/>(CTR, clicks, saves)"] --> ADLS
  TrendEngine --> Monitoring["MLflow + Azure Monitor<br/>(Drift, retraining triggers)"]
```

---

## 🔹 Detalle del Stack Propuesto

### Bronze Layer (Raw Ingest)

* **Azure Data Lake Storage Gen2 (ADLS)**: repositorio único para datos crudos.
* **Event Hubs / Kafka**: ingesta en tiempo real desde redes sociales y APIs.
* Fuentes: APIs de tendencias, ventas (Walmart/OpenFoodFacts), historial de recetas, dataset nutricional.

### Silver Layer (Cleansed / Structured)

* **Databricks Delta Live Tables**: limpieza, validación y estandarización.
* **Databricks Feature Store**: centraliza features (crecimiento, embeddings, nutrición).
* **Delta Lake (time-series)**: series temporales enriquecidas para modelado.

### Gold Layer (Business-Ready)

* **Trend Engine**: modelos de series temporales (Prophet, ARIMA), embeddings (Azure OpenAI), clustering (UMAP, HDBSCAN).
* **Candidate Generator**: RAG con Azure Cognitive Search + Azure OpenAI para generar recetas nuevas basadas en tendencias.
* **Ranking & Business Rules**: XGBoost/LightGBM + reglas de negocio (nutrición, estacionalidad, diversidad).
* **Serving**: Model Serving en Databricks + Azure Cache for Redis para baja latencia; API Management + CMS para exposición.

### Monitoring & Analytics

* **MLflow**: experiment tracking y versiones de modelos.
* **Azure Monitor + Evidently AI**: detección de *drift* y triggers de reentrenamiento.
* **Application Insights + Power BI**: métricas de performance online (CTR, guardados, uplift).

---

## 🔹 Plan de Experimentación

### Offline (validación inicial)

* **Precision@k / Recall@k**: exactitud de las recomendaciones.
* **Novelty / Diversity**: grado de innovación y variedad en las recetas.
* **NDCG (Normalized Discounted Cumulative Gain)**: ranking de relevancia.

### Online (A/B testing)

* **CTR uplift**: comparación entre recomendaciones actuales vs. modelo propuesto.
* **Engagement**: guardados, compartidos, comentarios.
* **Conversion**: adopción real de nuevas recetas sugeridas.

---

✅ Con esta arquitectura basada en Azure + Databricks Medallion se logra:

* Escalabilidad para streaming + batch.
* Reproducibilidad (Feature Store + MLflow).
* Creatividad controlada (RAG + LLMs).
* Ciclo cerrado de mejora continua con feedback de usuarios.
