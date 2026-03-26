# Deploying a Predictive Model: Diabetes Type 2 Risk Assessment

In this guide, we will deploy a predictive model designed to assess a patient's risk of developing Type 2 Diabetes based on clinical metrics (tabular data). 

Unlike Deep Learning models that process raw images or signals, tabular models (often built with Scikit-Learn, XGBoost, or LightGBM) process structured rows of data. 

The biggest challenge in deploying these models is **data validation**—ensuring the API receives the exact features, in the exact format, that the model expects. 

We will use **FastAPI** and **Pydantic** to enforce strict data rules.

---

## Prerequisites

Before starting, ensure you have:

* A trained model saved as a serialized file (e.g., `diabetes_rf_model.joblib` or a `.pkl` file).
* Docker installed locally.

---

## The Inference API (`app.py`)

We will use `Pydantic` to create a strict schema. If a client sends a request missing the `bmi` field, or sends a string instead of a float for `glucose`, FastAPI will automatically reject the request with a helpful error message before it ever reaches your model.

Create a file named `app.py`:

```python
import joblib
import numpy as np
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="Diabetes Risk Assessment API")

# 1. Define the strict input schema for your tabular data
class PatientData(BaseModel):
    pregnancies: int = Field(ge=0, description="Number of times pregnant")
    glucose: float = Field(ge=0, description="Plasma glucose concentration")
    blood_pressure: float = Field(ge=0, description="Diastolic blood pressure (mm Hg)")
    skin_thickness: float = Field(ge=0, description="Triceps skin fold thickness (mm)")
    insulin: float = Field(ge=0, description="2-Hour serum insulin (mu U/ml)")
    bmi: float = Field(ge=0, description="Body mass index (weight in kg/(height in m)^2)")
    diabetes_pedigree: float = Field(ge=0, description="Diabetes pedigree function")
    age: int = Field(ge=0, description="Age in years")

# 2. Load the Scikit-Learn model globally
try:
    # Replace with your actual model filename
    model = joblib.load('diabetes_rf_model.joblib')
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

@app.post("/predict")
def predict_risk(patient: PatientData):
    if model is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    try:
        # Convert the validated Pydantic object into a 2D NumPy array
        # Order must match the exact column order used during model training!
        features = np.array([[
            patient.pregnancies,
            patient.glucose,
            patient.blood_pressure,
            patient.skin_thickness,
            patient.insulin,
            patient.bmi,
            patient.diabetes_pedigree,
            patient.age
        ]])
        
        # Run inference
        prediction = model.predict(features)[0]
        probabilities = model.predict_proba(features)[0]
        
        # Assuming class 1 is "High Risk" and class 0 is "Low Risk"
        risk_level = "High Risk" if prediction == 1 else "Low Risk"
        confidence = probabilities[1] if prediction == 1 else probabilities[0]

        return {
            "risk_assessment": risk_level,
            "confidence": round(float(confidence), 4),
            "model_version": "1.0"
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": model is not None}
```

## Managing Dependencies (`requirements.txt`)
Tabular models are incredibly lightweight. Create your requirements.txt:

```python
Plaintext
fastapi==0.103.2
uvicorn==0.23.2
pydantic==2.4.2
scikit-learn==1.3.1
joblib==1.3.2
numpy==1.26.0
```

> **Info**: Ensure the version of `scikit-learn` in your `requirements.txt` exactly matches the version you used to train and save the `.joblib` file. Mismatched versions are the #1 cause of serialization errors in tabular deployments.

## The Dockerfile
Because we aren't loading heavy deep learning libraries like PyTorch or TensorFlow, this Docker image will be extremely small and fast to build.

```python
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the API code and the .joblib model file
COPY . .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### The `.dockerignore` file

As always, keep your containers clean:

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
Tabular model containers start up almost instantly, making them perfect for auto-scaling on our platform.

Build the Docker Image:

```python
docker build -t your-registry/diabetes-risk-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/diabetes-risk-api:v1
```

Deploy on the Platform: 

* Visit Crane Cloud and create a project to deploy the image `your-registry/diabetes-risk-api:v1` 
* Because this container is so lightweight, we can have aggressive auto-scaling rules (e.g., scaling from 1 to 10 pods during high-traffic periods) without worrying about slow cold-start times.

## Testing the Endpoint
Test your API by sending a JSON payload that perfectly matches your Pydantic schema:

```python
curl -X POST "https://diabetes-risk-api.ahumain.cranecloud.io/predict" \
  -H "Content-Type: application/json" \
  -d '{
        "pregnancies": 2,
        "glucose": 138.0,
        "blood_pressure": 62.0,
        "skin_thickness": 35.0,
        "insulin": 0.0,
        "bmi": 33.6,
        "diabetes_pedigree": 0.127,
        "age": 47
      }'
```

### Expected Response

```python
{
  "risk_assessment": "High Risk",
  "confidence": 0.8421,
  "model_version": "1.0"
}
```