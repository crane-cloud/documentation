# Deploying Cybersecurity Models: Financial Fraud Detection

In this guide, we will deploy a machine learning model designed to detect fraudulent financial transactions in real-time. 

Fraud detection models (typically built using XGBoost, Random Forests, or Isolation Forests) process tabular data representing a single transaction. The critical challenge here is **speed and reliability**.

This API must be lightweight enough to respond in milliseconds and robust enough to reject malformed data without crashing. We will use **FastAPI** combined with **Pydantic** to enforce strict data validation.

---

## Prerequisites

Before starting, ensure you have:

* A trained, serialized model file (e.g., `fraud_model.joblib` or `xgboost_fraud.pkl`).
* Docker installed locally.

---

## The Inference API (`app.py`)

Financial data must be validated strictly. If a transaction arrives with a negative amount or a missing merchant ID, our API should reject it instantly before it reaches the model.

Create a file named `app.py`:

```python
import joblib
import numpy as np
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="Real-Time Fraud Detection API")

# 1. Define a strict schema for the incoming transaction
class TransactionData(BaseModel):
    transaction_id: str = Field(..., description="Unique identifier for the transaction")
    amount: float = Field(gt=0, description="Transaction amount in USD (must be positive)")
    merchant_category_code: int = Field(..., description="MCC of the merchant")
    customer_age: int = Field(ge=18, description="Age of the customer")
    distance_from_home: float = Field(ge=0, description="Distance from home address in miles")
    is_online_transaction: int = Field(ge=0, le=1, description="1 if online, 0 if in-store")

# 2. Load the model globally to ensure low latency per request
try:
    # Replace with your actual model filename
    model = joblib.load('fraud_model.joblib')
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

@app.post("/score-transaction")
def score_transaction(transaction: TransactionData):
    if model is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    try:
        # Extract features in the exact order the model expects
        features = np.array([[
            transaction.amount,
            transaction.merchant_category_code,
            transaction.customer_age,
            transaction.distance_from_home,
            transaction.is_online_transaction
        ]])
        
        # Run inference
        # Fraud models often use predict_proba or an anomaly score
        probabilities = model.predict_proba(features)[0]
        fraud_probability = float(probabilities[1]) # Assuming class 1 is Fraud
        
        # Define your business logic threshold (e.g., > 85% is flagged)
        is_flagged = fraud_probability > 0.85

        return {
            "transaction_id": transaction.transaction_id,
            "fraud_probability": round(fraud_probability, 4),
            "action": "DECLINE" if is_flagged else "APPROVE"
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": model is not None}
```

## Managing Dependencies (`requirements.txt`)
Tabular ML environments are typically very lean. Create your requirements.txt:

```python
fastapi==0.103.2
uvicorn==0.23.2
pydantic==2.4.2
scikit-learn==1.3.1
joblib==1.3.2
numpy==1.26.0
```

> **Important**: Always ensure the version of `scikit-learn` (or `xgboost`) in your `requirements.txt` matches the version used to train the model. Version mismatches will cause fatal errors when the API attempts to load the `.joblib file`.

## The Dockerfile
Because latency is critical, we want the smallest, fastest-booting container possible.

```python
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code and model weights
COPY . .

EXPOSE 8000

# Run with multiple workers to handle concurrent transactions
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

> **Pro Tip**: Notice the `--workers` 4 flag in the `CMD` instruction. This tells Uvicorn to spin up multiple concurrent worker processes, drastically increasing the number of transactions your container can handle simultaneously.

### The `.dockerignore` file

```python
__pycache__/
*.pyc
.venv/
venv/
.git/
*.ipynb
data/
*.csv
```

## Deployment Steps
Fraud detection systems require high availability.

Build the Docker Image:

```python
docker build -t your-registry/fraud-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/fraud-api:v1
```

Deploy on the Platform:

 * Visit Crane Cloud and create a project to deploy the image `your-registry/fraud-api:v1` 
 * Auto-Scaling: Because transaction volumes are bursty (e.g., Black Friday sales), Crane Cloud handles horizontal auto-scaling for you.

## Testing the Endpoint
Test your real-time scoring API by sending a mock transaction payload:

```python
curl -X POST "https://fraud-api.ahumain.cranecloud.io/score-transaction" \
  -H "Content-Type: application/json" \
  -d '{
        "transaction_id": "TXN-99827364",
        "amount": 4500.00,
        "merchant_category_code": 5942,
        "customer_age": 24,
        "distance_from_home": 1250.5,
        "is_online_transaction": 1
      }'
```

### Expected Response

```python
{
  "transaction_id": "TXN-99827364",
  "fraud_probability": 0.9214,
  "action": "DECLINE"
}
```