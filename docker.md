# Docker Production-Level MLOps Project with FastAPI

## Objective

Build a production-ready Machine Learning application using:

- Python
- Scikit-Learn
- FastAPI
- Docker
- Docker Compose
- Persistent Docker Volumes
- REST API
- curl Testing

This project demonstrates how ML models are trained, containerized, served through APIs, and tested using curl requests.

---

# Architecture

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Python ML Application
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
REST API
    │
    ▼
curl / Postman
```

---

# Project Structure

```text
docker-mlops-lab/
│
├── data/
│   └── house_prices.csv
│
├── models/
│
├── src/
│   ├── train.py
│   └── app.py
│
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
│
└── README.md
```

---

# Dataset

## data/house_prices.csv

```csv
area,bedrooms,bathrooms,age,price
1200,2,2,10,4500000
1500,3,2,5,6000000
1800,3,3,4,7500000
2200,4,3,2,9000000
2500,4,4,1,11000000
1300,2,2,8,5000000
1600,3,2,6,6500000
2000,3,3,3,8000000
2400,4,3,2,10000000
2800,5,4,1,12500000
```

---

# Step 1: Install Docker

```bash
sudo apt update

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker
```

Verify:

```bash
docker --version
```

---

# Step 2: Install Docker Compose

```bash
sudo apt install docker-compose -y

docker-compose --version
```

---

# Step 3: Create Project

```bash
mkdir docker-mlops-lab

cd docker-mlops-lab
```

---

# Step 4: Create Project Structure

```bash
mkdir -p data

mkdir -p src

mkdir -p models
```

---

# Step 5: Create Requirements File

## requirements.txt

```txt
pandas
numpy
scikit-learn
joblib
fastapi
uvicorn
```

---

# Step 6: Create Training Script

## src/train.py

```python
import pandas as pd

from sklearn.ensemble import RandomForestRegressor

import joblib

df = pd.read_csv("data/house_prices.csv")

X = df.drop("price", axis=1)

y = df["price"]

model = RandomForestRegressor(
    n_estimators=100,
    random_state=42
)

model.fit(X, y)

joblib.dump(
    model,
    "models/model.pkl"
)

print("Model Saved Successfully")
```

---

# Step 7: Train Model Locally

```bash
python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt

python src/train.py
```

Verify:

```bash
ls models
```

Expected:

```text
model.pkl
```

---

# Step 8: Create FastAPI Application

## src/app.py

```python
from fastapi import FastAPI

import joblib

import pandas as pd

app = FastAPI()

model = joblib.load("models/model.pkl")

@app.get("/")
def health():

    return {
        "status": "healthy"
    }

@app.post("/predict")
def predict(data: dict):

    df = pd.DataFrame([data])

    prediction = model.predict(df)

    return {
        "prediction": float(prediction[0])
    }
```

---

# Step 9: Test FastAPI Locally

```bash
uvicorn src.app:app \
--host 0.0.0.0 \
--port 8000
```

Open:

```text
http://localhost:8000/docs
```

---

# Step 10: Create Dockerfile

## Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install \
--no-cache-dir \
-r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn","src.app:app","--host","0.0.0.0","--port","8000"]
```

---

# Step 11: Create Docker Ignore

## .dockerignore

```text
venv
.git
__pycache__
*.pyc
```

---

# Step 12: Build Docker Image

```bash
docker build -t house-price-api .
```

Verify:

```bash
docker images
```

---

# Step 13: Run Container

```bash
docker run -d \
--name house-price-api \
-p 8000:8000 \
house-price-api
```

Verify:

```bash
docker ps
```

---

# Step 14: Create Persistent Volume

```bash
docker volume create ml-models
```

Verify:

```bash
docker volume ls
```

---

# Step 15: Run Container with Persistent Volume

```bash
docker rm -f house-price-api

docker run -d \
--name house-price-api \
-p 8000:8000 \
-v ml-models:/app/models \
house-price-api
```

Benefits:

```text
Model Persistence
No Data Loss
Container Recreation Safe
```

---

# Step 16: Create Docker Compose

## docker-compose.yml

```yaml
version: "3.9"

services:

  ml-app:

    build: .

    container_name: house-price-api

    ports:
      - "8000:8000"

    restart: always

    volumes:
      - ml-models:/app/models

volumes:

  ml-models:
```

---

# Step 17: Deploy Using Docker Compose

```bash
docker-compose up -d
```

Verify:

```bash
docker ps
```

---

# Step 18: Health Check API Test

```bash
curl http://localhost:8000
```

Expected Output:

```json
{
  "status":"healthy"
}
```

---

# Step 19: Prediction API Test

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

Expected Output:

```json
{
  "prediction": 6950000.34
}
```

---

# Step 20: Load Testing

```bash
for i in {1..10}
do
curl -X POST \
http://localhost:8000/predict \
-H "Content-Type: application/json" \
-d '{
  "area":2000,
  "bedrooms":3,
  "bathrooms":3,
  "age":3
}'
echo ""
done
```

---

# Step 21: View Container Logs

```bash
docker logs -f house-price-api
```

---

# Step 22: Restart Container Test

```bash
docker restart house-price-api
```

Verify:

```bash
curl http://localhost:8000
```

Application should still work.

---

# Step 23: Delete Container Test

```bash
docker rm -f house-price-api
```

Recreate:

```bash
docker-compose up -d
```

Data remains because Docker Volume persists.

---

# Step 24: Backup Docker Volume

Backup:

```bash
docker run \
--rm \
-v ml-models:/source \
-v $(pwd):/backup \
ubuntu \
tar czf \
/backup/models.tar.gz \
-C /source .
```

Restore:

```bash
docker run \
--rm \
-v ml-models:/target \
-v $(pwd):/backup \
ubuntu \
bash -c \
"cd /target && tar xzf /backup/models.tar.gz"
```

---

# Production Best Practices

## Docker

```text
Use Slim Images
Use Multi-Stage Builds
Use Docker Compose
Use Volumes
Use Health Checks
Use Resource Limits
```

## Security

```text
No Hardcoded Secrets
Use Environment Variables
Run Non-Root User
Scan Images
```

## Reliability

```text
Restart Policies
Persistent Volumes
Backup Strategy
Container Monitoring
```

---

# Enterprise MLOps Architecture

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
Docker Build
   │
   ▼
Docker Registry
   │
   ▼
Kubernetes
   │
   ▼
FastAPI
   │
   ▼
KServe
   │
   ▼
Prometheus
   │
   ▼
Grafana
```

---

# Skills Covered

- Docker
- Docker Compose
- Docker Volumes
- Docker Networking
- FastAPI
- REST API Development
- curl Testing
- Model Serving
- Container Persistence
- Backup & Recovery
- Production Deployment
- MLOps Fundamentals

---

# Suggested Repository Names

```text
docker-mlops-lab
enterprise-docker-mlops
docker-model-serving-platform
production-docker-ml-platform
fastapi-docker-mlops
```

## Recommended

```text
enterprise-docker-mlops
```
