# Deploying a Signal Processing Model: EEG Classification

In this guide, we will deploy a machine learning model designed to analyze Electroencephalogram (EEG) signals. This specific model classifies the signals into three categories: Control, Alzheimer’s Disease (AD), and Frontotemporal Dementia (FTD).

Unlike computer vision which uses images, signal processing models (often 1D-CNNs or LSTMs) typically ingest continuous time-series data. 

We will build a **FastAPI** application that accepts raw EEG data arrays via a JSON payload, processes the signal, and runs inference using **PyTorch**.

---

## Prerequisites

Before starting, ensure you have:

* A trained PyTorch model weights file (e.g., `eeg_model.pt`).
* Docker installed locally.

---

## The Inference API (`app.py`)

For signal data, we use Pydantic to validate the incoming JSON payload. This ensures the API only accepts properly formatted arrays of numbers before attempting to run complex math operations on them.

Create a file named `app.py`:

```python
import numpy as np
import torch
import torch.nn as nn
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(title="EEG Classification API")

# 1. Define the input data structure using Pydantic
class EEGPayload(BaseModel):
    # Expecting a list of floats representing a time-series window
    signal: list[float]
    patient_id: str | None = None

# 2. Define your model architecture (must match your saved weights)
class EEGClassifier(nn.Module):
    def __init__(self):
        super(EEGClassifier, self).__init__()
        # Example 1D CNN architecture
        self.conv1 = nn.Conv1d(in_channels=1, out_channels=16, kernel_size=3)
        self.fc = nn.Linear(16 * 254, 3) # Simplified for example

    def forward(self, x):
        x = torch.relu(self.conv1(x))
        x = x.view(x.size(0), -1)
        return self.fc(x)

# 3. Load the model globally
device = torch.device("cpu") # Force CPU for lightweight deployment
model = EEGClassifier()
try:
    model.load_state_dict(torch.load('eeg_model.pt', map_location=device))
    model.eval()
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

CLASS_NAMES = ["Control", "Alzheimer's Disease (AD)", "Frontotemporal Dementia (FTD)"]

def preprocess_signal(raw_signal: list[float]) -> torch.Tensor:
    """Normalizes and reshapes the 1D signal for PyTorch."""
    signal_array = np.array(raw_signal)
    
    # Standard normalization (Z-score)
    mean = np.mean(signal_array)
    std = np.std(signal_array)
    normalized = (signal_array - mean) / (std + 1e-8)
    
    # Reshape to (Batch, Channels, Sequence_Length) -> (1, 1, 256)
    tensor = torch.tensor(normalized, dtype=torch.float32).unsqueeze(0).unsqueeze(0)
    return tensor

@app.post("/predict")
def predict_eeg(payload: EEGPayload):
    if model is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    # Validate signal length (e.g., expecting 256 data points)
    if len(payload.signal) != 256:
        raise HTTPException(status_code=400, detail="Signal must contain exactly 256 data points.")

    try:
        # Preprocess and predict
        input_tensor = preprocess_signal(payload.signal)
        
        with torch.no_grad():
            outputs = model(input_tensor)
            probabilities = torch.softmax(outputs, dim=1)[0]
            predicted_idx = torch.argmax(probabilities).item()
            confidence = probabilities[predicted_idx].item()

        return {
            "patient_id": payload.patient_id,
            "diagnosis": CLASS_NAMES[predicted_idx],
            "confidence": round(confidence, 4)
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": model is not None}
```

## Managing Dependencies (`requirements.txt`)

Create a `requirements.txt` file.

```python
fastapi==0.103.2
uvicorn==0.23.2
pydantic==2.4.2
numpy==1.26.0
```

> **Crucial PyTorch Info**: Notice that `torch` is not in the `requirements.txt`. By default, `pip install torch` downloads massive CUDA (GPU) libraries, resulting in a 4GB+ Docker image. We will handle installing the CPU-only version directly in the Dockerfile to keep the container under 500MB.

## The Dockerfile

This Dockerfile is specifically optimized for PyTorch CPU deployments.

```python
FROM python:3.10-slim

WORKDIR /app

# Install standard requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Install PyTorch CPU-only version explicitly to save gigabytes of space
RUN pip install --no-cache-dir torch==2.1.0+cpu --index-url https://download.pytorch.org/whl/cpu

# Copy application and model files
COPY . .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### The `.dockerignore` file

Ensure you don't copy local environments or training data:

```python
__pycache__/
*.pyc
.venv/
venv/
.git/
*.ipynb
data/
```

## Deployment Steps
Deploying to the cluster follows the standard pipeline:

Build the Docker Image:

```python
docker build -t your-registry/eeg-classifier-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/eeg-classifier-api:v1
```

Deploy on the Platform: 

* Visit Crane Cloud and create a project to use the image `your-registry/eeg-classifier-api:v1` 
* Set the Health Check to the `/health` endpoint to ensure traffic only routes when PyTorch has successfully initialized.

## Testing the Endpoint
Once deployed, test your API using a JSON payload containing an array of floats.

```python
curl -X POST "https://eeg-classifier-api.ahumain.cranecloud.io/predict" \
  -H "Content-Type: application/json" \
  -d '{
        "patient_id": "PT-8842",
        "signal": [0.12, -0.45, 0.89, -0.11, /* ... imagine 252 more floats here ... */ 0.33]
      }'
```

### Expected Response

```python
{
  "patient_id": "PT-8842",
  "diagnosis": "Control",
  "confidence": 0.9214
}
```