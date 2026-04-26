# SentimentOps-AI

End-to-end MLOps capstone project for sentiment analysis on IMDB-style movie reviews.

This repository covers the full lifecycle:
- data ingestion and preprocessing
- feature engineering with Bag-of-Words
- model training and evaluation
- MLflow + DagsHub tracking and model registration
- Flask web app inference endpoint
- containerization and Kubernetes deployment
- basic model and app tests

## Project Highlights

- Modular pipeline under `src/` (data, features, model, visualization)
- Reproducible stages defined in `dvc.yaml`
- Config-driven hyperparameters in `params.yaml`
- Model registry integration using MLflow and DagsHub
- Production-oriented serving app with Prometheus metrics (`/metrics`)
- Deployment assets: `Dockerfile` and `deployment.yaml`

## Repository Structure

```text
.
|- dvc.yaml
|- params.yaml
|- Dockerfile
|- deployment.yaml
|- flask_app/
|  |- app.py
|  |- requirements.txt
|  `- templates/
|- src/
|  |- data/
|  |- features/
|  |- model/
|  `- visualization/
|- tests/
|  |- test_flask_app.py
|  `- test_model.py
|- notebooks/
`- reports/
```

## ML Pipeline

Pipeline stages are defined in `dvc.yaml`:
1. `data_ingestion` -> loads raw data into `data/raw`
2. `data_preprocessing` -> writes cleaned splits into `data/interim`
3. `feature_engineering` -> creates BoW features and `models/vectorizer.pkl`
4. `model_building` -> trains and saves `models/model.pkl`
5. `model_evaluation` -> writes `reports/metrics.json` and run metadata
6. `model_registration` -> registers model in MLflow Model Registry

Tunable parameters (from `params.yaml`):
- `data_ingestion.test_size`
- `feature_engineering.max_features`

## Prerequisites

- Python 3.10+ recommended
- pip
- (Optional) DVC for pipeline orchestration
- DagsHub token for MLflow tracking-based stages and app runtime

## Setup

### 1) Clone and create virtual environment

```bash
git clone <your-fork-or-repo-url>
cd YT-Capstone-Project

# Windows (PowerShell)
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Linux / macOS
python -m venv .venv
source .venv/bin/activate
```

### 2) Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## Configure Environment

Several scripts and the Flask app require `CAPSTONE_TEST` for DagsHub/MLflow auth.

### Windows (PowerShell)

```powershell
$env:CAPSTONE_TEST="<your_dagshub_token>"
```

### Linux / macOS

```bash
export CAPSTONE_TEST="<your_dagshub_token>"
```

## Run the Pipeline

### Option A: Run full pipeline with DVC

```bash
dvc repro
```

### Option B: Run stage scripts manually

```bash
python src/data/data_ingestion.py
python src/data/data_preprocessing.py
python src/features/feature_engineering.py
python src/model/model_building.py
python src/model/model_evaluation.py
python src/model/register_model.py
```

## Run the Flask App

```bash
cd flask_app
python app.py
```

App endpoints:
- `GET /` -> UI form for sentiment prediction
- `POST /predict` -> predicts sentiment from submitted text
- `GET /metrics` -> Prometheus-format custom metrics

Default local URL:
- `http://localhost:5000`

## Docker

Build image from repository root:

```bash
docker build -t yt-capstone:latest .
```

Run container:

```bash
docker run --rm -p 5000:5000 -e CAPSTONE_TEST=<your_dagshub_token> yt-capstone:latest
```

The container starts the app with Gunicorn (`app:app`) on port `5000`.

## Kubernetes

Kubernetes deployment and service are defined in `deployment.yaml`.

Apply manifests:

```bash
kubectl apply -f deployment.yaml
```

Notes:
- Deployment uses 2 replicas of the Flask app.
- Service type is `LoadBalancer` on port `5000`.
- Secret key expected: `capstone-secret` with key `CAPSTONE_TEST`.

## Testing

Run tests from repo root:

```bash
python -m unittest tests/test_flask_app.py
python -m unittest tests/test_model.py
```

Or discover all tests:

```bash
python -m unittest discover -s tests
```

## Documentation

Sphinx docs skeleton is available under `docs/`.

Build docs:

```bash
cd docs
make html
```

## Troubleshooting

- `CAPSTONE_TEST environment variable is not set`
	- Export the token before running MLflow-connected scripts or `flask_app/app.py`.
- `vectorizer.pkl not found`
	- Ensure `feature_engineering` stage has run and generated `models/vectorizer.pkl`.
- `model not found in registry`
	- Run training/evaluation/registration stages first, then restart the app.

## Acknowledgment

Built as an educational MLOps capstone to demonstrate an end-to-end production-style ML workflow.
