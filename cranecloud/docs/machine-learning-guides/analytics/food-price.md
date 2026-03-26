# Deploying Predictive Analytics: Food Price Forecasting

In this guide, we will deploy a time-series forecasting model designed to predict agricultural commodities and food prices.

Time-series models (whether traditional like ARIMA/Prophet, or machine learning models using lag features like XGBoost) differ from standard predictive models because they rely on **historical context**. To predict tomorrow's price, the API cannot just receive static features; it must receive a sequence (or "window") of past prices. 

We will build a **FastAPI** application that accepts an array of historical data points and returns the forecasted future value.

---

## Prerequisites

Before starting, ensure you have:

* A trained time-series model saved as a serialized file (e.g., `wheat_price_model.joblib`).
* Docker installed locally.
* An understanding of the "lookback window" your model requires (e.g., it needs exactly 30 days of past data to make a prediction).

---

## The Inference API (`app.py`)

We will use Pydantic to strictly enforce the length of the historical data array. If your model was trained on a 14-day lookback window, passing 13 or 15 days will cause it to crash. Pydantic prevents this before the data ever touches the model.

Create a file named `app.py`:

```python
import joblib
import numpy as np
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="Food Price Forecasting API")

# Define the expected lookback window (e.g., 14 days of historical prices)
LOOKBACK_WINDOW = 14

# 1. Define the schema for the incoming historical data
class TimeSeriesData(BaseModel):
    commodity: str = Field(..., description="Name of the crop or commodity (e.g., 'Wheat', 'Corn')")
    historical_prices: list[float] = Field(
        ..., 
        description=f"An array of exactly {LOOKBACK_WINDOW} historical daily prices."
    )

# 2. Load the forecasting model globally
try:
    # Replace with your actual model filename
    model = joblib.load('wheat_price_model.joblib')
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

@app.post("/forecast")
def forecast_price(data: TimeSeriesData):
    if model is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    # 3. Strictly validate the window size
    if len(data.historical_prices) != LOOKBACK_WINDOW:
        raise HTTPException(
            status_code=400, 
            detail=f"Expected exactly {LOOKBACK_WINDOW} data points, got {len(data.historical_prices)}."
        )

    try:
        # Reshape the 1D list into a 2D array for Scikit-Learn/XGBoost
        features = np.array([data.historical_prices])
        
        # Run inference to predict the next time step (e.g., day 15)
        predicted_price = float(model.predict(features)[0])

        return {
            "commodity": data.commodity,
            "forecasted_price": round(predicted_price, 2),
            "currency": "USD",
            "model_version": "1.0"
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": model is not None}
```

## Managing Dependencies (`requirements.txt`)
Create your requirements.txt:

```python
fastapi==0.103.2
uvicorn==0.23.2
pydantic==2.4.2
scikit-learn==1.3.1
joblib==1.3.2
numpy==1.26.0
```

## The Dockerfile
Because this is a standard tabular/numerical model, the Dockerfile remains highly optimized and lightweight.

```python
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code and model weights
COPY . .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

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
Time-series forecasting models often have unique usage patterns on our platform.

Build the Docker Image:

```python
docker build -t your-registry/food-price-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/food-price-api:v1
```

Deploy on the Platform:

* Visit Crane Cloud and create a project to deploy the image `your-registry/food-price-api:v1` 
* Scale to Zero: Because this API might only be hit a few times a day, Crane Cloud is able to scale this deployment down to zero pods when inactive to save heavily on cloud costs.

## Testing the Endpoint
Test your API by sending a JSON payload containing the required array of historical prices:

```python
curl -X POST "https://food-price-api.ahumain.cranecloud.io/forecast" \
  -H "Content-Type: application/json" \
  -d '{
        "commodity": "Wheat",
        "historical_prices": [
          210.5, 212.0, 211.5, 209.0, 208.5, 210.0, 213.5, 
          215.0, 216.5, 218.0, 217.5, 219.0, 220.5, 222.0
        ]
      }'
```

### Expected Response

```python
{
  "commodity": "Wheat",
  "forecasted_price": 224.3,
  "currency": "USD",
  "model_version": "1.0"
}
```