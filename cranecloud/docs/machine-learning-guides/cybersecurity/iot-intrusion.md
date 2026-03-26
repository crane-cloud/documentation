# Deploying Cybersecurity Models: IoT Intrusion Detection

In this guide, we will deploy a machine learning model designed to monitor network traffic from Internet of Things (IoT) devices and detect malicious intrusions or anomalies (such as DDoS attacks or unauthorized access attempts).

IoT deployments generate massive amounts of high-frequency telemetry data. The model (often an unsupervised anomaly detector like an Isolation Forest or One-Class SVM) must be able to score incoming network packets instantly.

We will build and deploy a highly concurrent **FastAPI** application capable of handling rapid, continuous requests from your IoT gateway.

---

## Prerequisites

Before starting, ensure you have:
* A trained anomaly detection model saved as a serialized file (e.g., `iot_isolation_forest.joblib`).
* Docker installed locally.

---

## The Inference API (`app.py`)

Our API needs to accept network packet metadata. Since IoT devices often send data in bursts, we will design our endpoint to accept a single packet reading, but you can easily scale this to accept batches (lists) of readings.

Create a file named `app.py`:

```python
import joblib
import numpy as np
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="IoT Intrusion Detection API")

# 1. Define the schema for IoT network telemetry
class NetworkTelemetry(BaseModel):
    device_id: str = Field(..., description="Unique ID of the IoT device")
    packet_size: float = Field(ge=0, description="Size of the network packet in bytes")
    connection_duration: float = Field(ge=0, description="Duration of the connection in seconds")
    failed_logins: int = Field(ge=0, description="Number of failed login attempts")
    bytes_sent: float = Field(ge=0, description="Total bytes sent by the device")
    bytes_received: float = Field(ge=0, description="Total bytes received by the device")

# 2. Load the anomaly detection model globally
try:
    # Replace with your actual model filename
    model = joblib.load('iot_isolation_forest.joblib')
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

@app.post("/detect-anomaly")
def detect_intrusion(telemetry: NetworkTelemetry):
    if model is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    try:
        # Extract features into a 2D array
        features = np.array([[
            telemetry.packet_size,
            telemetry.connection_duration,
            telemetry.failed_logins,
            telemetry.bytes_sent,
            telemetry.bytes_received
        ]])
        
        # Run inference
        # Isolation Forest typically returns -1 for anomalies/outliers and 1 for inliers
        prediction = model.predict(features)[0]
        
        is_anomaly = True if prediction == -1 else False
        
        # Some Scikit-Learn anomaly models also support decision_function for an anomaly score
        anomaly_score = float(model.decision_function(features)[0])

        return {
            "device_id": telemetry.device_id,
            "is_intrusion_detected": is_anomaly,
            "threat_score": round(anomaly_score, 4),
            "action": "QUARANTINE_DEVICE" if is_anomaly else "ALLOW"
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

> **Reminder**: Ensure your `scikit-learn` version exactly matches the environment where the model was trained to avoid serialization errors!

## The Dockerfile
To handle the high frequency of IoT telemetry, we will configure Uvicorn to run with multiple worker processes, allowing the container to process multiple network packets simultaneously.

```python
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code and model weights
COPY . .

EXPOSE 8000

# Use --workers to handle concurrent IoT data streams efficiently
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
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
*.pcap
```

## Deployment Steps
IoT intrusion detection APIs are the backbone of secure networks.

Build the Docker Image:

```python
docker build -t your-registry/iot-intrusion-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/iot-intrusion-api:v1
```

Deploy on the Platform:

 * Visit Crane Cloud and create a project to deploy the image `your-registry/iot-intrusion-api:v1` 
 * Readiness Probes: Set the health check to `/health`.

## Testing the Endpoint
Test your API by sending a JSON payload representing a suspicious network packet (e.g., high failed logins and massive data transfer in a short duration):

```python
curl -X POST "https://iot-intrusion-api.ahumain.cranecloud.io/detect-anomaly" \
  -H "Content-Type: application/json" \
  -d '{
        "device_id": "smart-thermostat-b8:27:eb:14:52:99",
        "packet_size": 1500.0,
        "connection_duration": 0.5,
        "failed_logins": 12,
        "bytes_sent": 50000.0,
        "bytes_received": 1024.0
      }'
```

### Expected Response

```python
{
  "device_id": "smart-thermostat-b8:27:eb:14:52:99",
  "is_intrusion_detected": true,
  "threat_score": -0.8543,
  "action": "QUARANTINE_DEVICE"
}
```