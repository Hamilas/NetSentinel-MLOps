# NetSentinel-MLOps

<p align="center">
  <img src="https://img.shields.io/badge/Random%20Forest-phishing%20detection-6366f1?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/MLflow-tracking-0194E2?style=for-the-badge&logo=mlflow&logoColor=white"/>
  <img src="https://img.shields.io/badge/FastAPI-inference-009688?style=for-the-badge&logo=fastapi&logoColor=white"/>
  <img src="https://img.shields.io/badge/Prometheus%20+%20Grafana-monitoring-E6522C?style=for-the-badge&logo=prometheus&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge"/>
</p>

<p align="center">
  <strong>AI-powered phishing URL detection pipeline. Random Forest with full MLOps instrumentation</strong><br/>
  SHAP explainability · MLflow tracking · Prometheus monitoring · real-time URL classification
</p>

<p align="center">
  <img src="assets/banner.svg?v=2" alt="NetSentinel-MLOps Banner" width="800"/>
</p>

> AI-powered phishing URL detection pipeline. Classifies URLs as PHISHING or LEGITIMATE using a Random Forest ensemble with full MLOps instrumentation, real-time monitoring, and explainable predictions.

## Live Demo

**Live:** [https://netsentinel-mlops-demo.vercel.app](https://netsentinel-mlops-demo.vercel.app)

## Screenshots

<table align="center">
  <tr>
    <td align="center" width="50%"><sub>Dashboard, real model evaluation and feature importances</sub></td>
    <td align="center" width="50%"><sub>Classify, real inference with explainable signal breakdown</sub></td>
  </tr>
  <tr>
    <td align="center" width="50%"><img src="assets/screenshots/dashboard.png" width="380"/></td>
    <td align="center" width="50%"><img src="assets/screenshots/classify.png" width="380"/></td>
  </tr>
</table>

## Overview

End-to-end MLOps pipeline that classifies URLs as **PHISHING** or **LEGITIMATE** using a RandomForest classifier (16 estimators) trained on 11,055 URLs from the UCI Phishing Websites Detection dataset. Each URL is represented by 30 engineered security signals covering SSL, DNS, redirects, anchor links, traffic rank, and domain age.

The model achieves **98.7% accuracy** and **98.9% F1** on a held-out test split of 2,211 URLs, with sub-5ms inference latency per URL.

## Architecture

<p align="center">
  <img src="images/architecture.svg" alt="NetSentinel-MLOps Architecture" width="600"/>
</p>

## Tech Stack

| Technology | Purpose |
|---|---|
| Python 3.10 | Core language |
| FastAPI | Async REST API serving |
| scikit-learn | RandomForest, GridSearchCV, preprocessing |
| MLflow | Experiment tracking + model registry |
| MongoDB | Raw URL data ingestion |
| React 18 + Vite + React Router | 5-tab interactive SPA |
| Tailwind CSS + Recharts | Styling + charts |
| Docker Compose | Local orchestration, 3 services |
| scipy | Kolmogorov-Smirnov drift detection |

## Quick Start

```bash
git clone https://github.com/Hamilas/netsentinel-mlops
cd netsentinel-mlops
cp .env.example .env
docker compose up -d
```

Open:
- **React Dashboard:** http://localhost:8090/ui
- **MLflow UI:** http://localhost:5000
- **Swagger docs:** http://localhost:8090/docs

## API Reference

```bash
# Health check
curl http://localhost:8090/health

# Live model metrics (prediction stats from log)
curl http://localhost:8090/metrics

# Real model evaluation on held-out test split
curl http://localhost:8090/api/model-stats

# Single URL classification (30-feature vector)
curl -s -X POST http://localhost:8090/api/predict-single \
  -H "Content-Type: application/json" \
  -d '{"features": [-1,1,1,1,-1,-1,-1,-1,-1,1,1,-1,1,-1,1,-1,-1,-1,0,1,1,1,1,-1,-1,-1,-1,1,1,-1]}'
# → {"label":"LEGITIMATE","p_phishing":6.2,"p_legitimate":93.8,"latency_ms":4.3}

# Batch CSV prediction
curl -s -X POST http://localhost:8090/api/predict \
  -F "file=@valid_data/phishing_sample.csv"

# Recent predictions (live log)
curl http://localhost:8090/api/recent-predictions?limit=10

# 24-hour threat timeline (hourly buckets)
curl http://localhost:8090/api/timeline
```

## Dataset

- **Source:** UCI Phishing Websites Detection
- **Size:** 11,055 URLs, 6,157 phishing / 4,898 legitimate
- **Features:** 30 URL security signals (binary: 1 = suspicious, -1 = safe, 0 = neutral)
- **Top 3 features by model importance:** SSL Certificate (33.8%), Anchor Link Targets (21.1%), Web Traffic Rank (7.8%)
- **Labels:** 1 = PHISHING, −1 = LEGITIMATE (remapped to 0/1 internally)
- **Test split:** 20% held-out (2,211 URLs)

## Results

| Metric | Value |
|---|---|
| Accuracy | **98.7%** |
| F1 (weighted) | **98.7%** |
| F1 (phishing class) | **98.9%** |
| F1 (legitimate class) | **98.5%** |
| Precision | **98.7%** |
| Recall | **98.7%** |
| Avg inference latency | **< 5ms** per URL |
| Test set size | 2,211 URLs |
| Model | RandomForestClassifier (16 trees) |

## Dashboard Tabs

| Tab | What it shows |
|---|---|
| **Dashboard** | Live system stats, real model eval metrics, feature importance chart (from model.pkl), prediction split donut |
| **Batch Predict** | Upload CSV → classify all rows → color-coded results table with phishing rate |
| **Classify** | Upload CSV → click any row → live model inference with `predict_proba` confidence bars + plain-English signal breakdown |
| **Monitor** | 24h threat timeline (Recharts bar chart) + live prediction feed polling every 5s |
| **About** | Pipeline stages, tech stack, results, author |

## Features

- 5-algorithm ensemble with GridSearchCV, automatically selects the best model per training run
- Kolmogorov-Smirnov drift detection, auto-triggers retraining when input distribution shifts
- `/api/predict-single` endpoint with `predict_proba`, returns real model confidence (% of trees voting each class), not a heuristic score
- `/api/model-stats` evaluates the live model on a fresh test split on every call, returns real F1/accuracy/precision/recall
- Feature importance visualization from `model.pkl`, top 8 features ranked by real Gini importance
- React SPA with URL routing (`react-router-dom`), each tab has a bookmarkable URL
- Row-capped batch API (500-row preview) to prevent browser OOM on 11k-row CSVs
- Classify tab explains predictions in plain English: "SSL Certificate: Invalid/self-signed cert" instead of `SSLfinal_State: 1`

## European Market Use Cases

**German fintech / banking (N26, Deutsche Bank, Commerzbank):** Real-time phishing URL classification for transaction security and SOC email analysis pipelines.

**E-commerce (Zalando, Otto, Check24):** Automated screening of seller-submitted URLs and affiliate links before display to end users.

**Cybersecurity tooling (MSSPs):** On-premise phishing detection module that integrates into SIEM pipelines without data leaving the client network.

## Author

**Rayen Lassoued**
[github.com/Hamilas](https://github.com/Hamilas) | [https://www.linkedin.com/in/lassoued-rayen/](https://www.linkedin.com/in/lassoued-rayen/)

## License

MIT
