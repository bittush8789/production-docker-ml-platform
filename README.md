# Enterprise Docker MLOps Platform

## Overview

A production-ready MLOps project demonstrating how to train, containerize, and serve a Machine Learning model using **Docker**, **FastAPI**, and **Scikit-Learn**.

The project covers the complete workflow from model training to API-based inference, including persistent storage, container management, and production deployment practices.

## Features

* Machine Learning Model Training
* FastAPI-Based Model Serving
* Docker Containerization
* Docker Compose Deployment
* Persistent Docker Volumes
* REST API Endpoints
* curl-Based API Testing
* Production-Oriented Architecture

## Tech Stack

* Python
* Pandas
* NumPy
* Scikit-Learn
* FastAPI
* Uvicorn
* Docker
* Docker Compose
* Joblib

## Architecture

```text
Developer
    │
    ▼
GitHub
    │
    ▼
ML Training Pipeline
    │
    ▼
Docker Image
    │
    ▼
Docker Container
    │
    ▼
FastAPI
    │
    ▼
Prediction API
    │
    ▼
curl / Postman
```

## Project Structure

```text
enterprise-docker-mlops/
│
├── data/
│   └── house_prices.csv
│
├── models/
│   └── model.pkl
│
├── src/
│   ├── train.py
│   └── app.py
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .dockerignore
│
└── README.md
```

## Quick Start

### Clone Repository

```bash
git clone https://github.com/<your-username>/enterprise-docker-mlops.git

cd enterprise-docker-mlops
```

### Create Virtual Environment

```bash
python3 -m venv venv

source venv/bin/activate
```

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Train Model

```bash
python src/train.py
```

### Build Docker Image

```bash
docker build -t house-price-api .
```

### Run Container

```bash
docker run -d \
--name house-price-api \
-p 8000:8000 \
house-price-api
```

## API Endpoints

### Health Check

```http
GET /
```

### Prediction

```http
POST /predict
```

## Test Health Endpoint

```bash
curl http://localhost:8000
```

Response:

```json
{
  "status": "healthy"
}
```

## Test Prediction Endpoint

```bash
curl -X POST \
http://localhost:8000/predict \
-H "Content-Type: application/json" \
-d '{
  "area":1700,
  "bedrooms":3,
  "bathrooms":2,
  "age":4
}'
```

Response:

```json
{
  "prediction": 6950000.34
}
```

## Docker Compose Deployment

Start Services:

```bash
docker-compose up -d
```

Stop Services:

```bash
docker-compose down
```

View Logs:

```bash
docker logs -f house-price-api
```

## Production Features

* Persistent Docker Volumes
* Container Restart Support
* REST API Inference
* Automated Deployment with Docker Compose
* Portable and Reproducible Environment
* Production-Ready Containerization

## Skills Demonstrated

* Docker Fundamentals
* Docker Compose
* Container Lifecycle Management
* Volume Management
* FastAPI Development
* Model Serving
* REST APIs
* MLOps Fundamentals
* Production Deployment

## Future Enhancements

* GitHub Actions CI/CD
* MLflow Integration
* Kubernetes Deployment
* KServe Model Serving
* Prometheus Monitoring
* Grafana Dashboards
* Model Registry
* AWS ECR Integration

## License

This project is intended for learning and portfolio demonstration purposes.
