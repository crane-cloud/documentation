# Deploying NLP Models: Hugging Face API Wrapper

In this guide, we will deploy an open-source Natural Language Processing (NLP) model from Hugging Face. We will wrap a foundational model (in this example, a sentiment analysis model) into a production-ready API.

**The Golden Rule of Hugging Face Deployments:** Never configure your application to download a model from the internet when the container starts. Large models take minutes to download, which will cause your deployment to fail our platform's health checks. Instead, we will write a script to download the model *during the Docker build process* so it is permanently baked into your image.

---

## Prerequisites

Before starting, ensure you have:

* The name of the Hugging Face model you want to use (e.g., `distilbert-base-uncased-finetuned-sst-2-english`).
* Docker installed locally.

---

## The Model Downloader (`download_model.py`)

First, we write a standalone Python script whose sole purpose is to fetch the model and tokenizer from Hugging Face and save them to a local folder. 

Create a file named `download_model.py`:

```python
import os
from transformers import AutoModelForSequenceClassification, AutoTokenizer

# The Hugging Face model ID
MODEL_NAME = "distilbert-base-uncased-finetuned-sst-2-english"
# The local directory where we will save the weights
SAVE_DIR = "./model_weights"

def download_and_save():
    print(f"Downloading {MODEL_NAME}...")
    
    # Download tokenizer and model
    tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
    model = AutoModelForSequenceClassification.from_pretrained(MODEL_NAME)
    
    # Save them locally
    os.makedirs(SAVE_DIR, exist_ok=True)
    tokenizer.save_pretrained(SAVE_DIR)
    model.save_pretrained(SAVE_DIR)
    
    print(f"Model saved successfully to {SAVE_DIR}/")

if __name__ == "__main__":
    download_and_save()
```

## The Inference API (`app.py`)
Now we build the FastAPI application. Notice how we tell the transformers library to load the model from our local `./model_weights` directory instead of fetching it from the internet.

Create a file named `app.py`:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from transformers import pipeline

app = FastAPI(title="Hugging Face NLP API")

# 1. Define the expected incoming JSON payload
class TextPayload(BaseModel):
    text: str

# 2. Load the model from the LOCAL directory, not the internet
LOCAL_MODEL_DIR = "./model_weights"

try:
    # We use the pipeline abstraction for easy inference
    nlp_pipeline = pipeline(
        "sentiment-analysis", 
        model=LOCAL_MODEL_DIR, 
        tokenizer=LOCAL_MODEL_DIR
    )
except Exception as e:
    print(f"Failed to load model from {LOCAL_MODEL_DIR}. Error: {e}")
    nlp_pipeline = None

@app.post("/analyze")
def analyze_text(payload: TextPayload):
    if nlp_pipeline is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    if not payload.text.strip():
        raise HTTPException(status_code=400, detail="Text payload cannot be empty.")

    try:
        # Run the text through the Hugging Face pipeline
        result = nlp_pipeline(payload.text)[0]
        
        return {
            "input_text": payload.text,
            "label": result['label'],
            "score": round(result['score'], 4)
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": nlp_pipeline is not None}
```

## Managing Dependencies (`requirements.txt`)
We need FastAPI and the Hugging Face ecosystem. Create your requirements.txt:

```python
fastapi==0.103.2
uvicorn==0.23.2
pydantic==2.4.2
transformers==4.34.0
```

> **Note on PyTorch**: We will handle installing a lightweight, CPU-only version of PyTorch directly inside the Dockerfile to keep the image size manageable.

## The Dockerfile
This is the most important step. We will execute download_model.py as a RUN command inside the Dockerfile. This ensures the gigabytes of model weights become a permanent layer in your Docker image.

```python
FROM python:3.10-slim

WORKDIR /app

# 1. Install dependencies first
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 2. Install CPU-only PyTorch to save gigabytes of space
RUN pip install --no-cache-dir torch==2.1.0+cpu --index-url https://download.pytorch.org/whl/cpu

# 3. Copy the download script and execute it
# We do this BEFORE copying the rest of the app to leverage Docker caching
COPY download_model.py .
RUN python download_model.py

# 4. Copy the main API code
COPY app.py .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]

``` 

### The `.dockerignore` file

Ensure you do not accidentally copy local model weights if you ran the download script on your own laptop:

```python
__pycache__/
*.pyc
.venv/
venv/
model_weights/
```

## Deployment Steps
NLP models require careful resource allocation.

Build the Docker Image:

```python
docker build -t your-registry/hf-nlp-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/hf-nlp-api:v1
```

Deploy on the Platform:

 * Visit Crane Cloud and create a project to use the image `your-registry/hf-nlp-api:v1`
 * Memory Limits: Hugging Face models are RAM-heavy.

Readiness Probes: Set the health check to `/health`. Because the model weights are already baked into the image, the cold-start time will only be the few seconds it takes to load those weights from disk into memory.

## Testing the Endpoint
Send a JSON payload containing the text you want to analyze:

```python
curl -X POST "https://hf-nlp-api.ahumain.cranecloud.io/analyze" \
  -H "Content-Type: application/json" \
  -d '{
        "text": "Deploying models to this cloud native platform is incredibly smooth and easy!"
      }'
```

### Expected Response

```python
{
  "input_text": "Deploying models to this cloud native platform is incredibly smooth and easy!",
  "label": "POSITIVE",
  "score": 0.9987
}
```