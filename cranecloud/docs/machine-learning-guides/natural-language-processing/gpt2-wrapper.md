# Deploying Generative AI: GPT-2 Text Wrapper

In this guide, we will deploy a foundational Generative AI model. We are using **GPT-2** as our example because it is small enough to run comfortably on standard CPUs, making it perfect for learning how to deploy text-generation bots (like insurance guides, automated responders, or creative writing assistants).

Generative models differ from standard classifiers because they require specific parameters (like `temperature` and `max_length`) to control how creative and how long the generated response should be. We will expose these settings safely through our API.

---

## Prerequisites

Before starting, ensure you have:

* Docker installed locally.
* Basic familiarity with text generation concepts (prompts, temperature).

---

## The Model Downloader (`download_model.py`)

Just like with our Hugging Face classification guide, we **must not** download the model at runtime. We will write a script to fetch GPT-2 so we can bake it into our Docker image.

Create a file named `download_model.py`:

```python
import os
from transformers import AutoModelForCausalLM, AutoTokenizer

MODEL_NAME = "gpt2"
SAVE_DIR = "./model_weights"

def download_and_save():
    print(f"Downloading {MODEL_NAME}...")
    
    # Download tokenizer and causal language model
    tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
    model = AutoModelForCausalLM.from_pretrained(MODEL_NAME)
    
    # Save them locally
    os.makedirs(SAVE_DIR, exist_ok=True)
    tokenizer.save_pretrained(SAVE_DIR)
    model.save_pretrained(SAVE_DIR)
    
    print(f"GPT-2 saved successfully to {SAVE_DIR}/")

if __name__ == "__main__":
    download_and_save()
```

## The Inference API (`app.py`)
Our FastAPI application will use `Pydantic` to not only accept a text prompt, but to optionally accept generation parameters with safe default values.

Create a file named `app.py`:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
from transformers import pipeline

app = FastAPI(title="GPT-2 Text Generation API")

# 1. Define the incoming payload with safe default parameters
class GenerationRequest(BaseModel):
    prompt: str = Field(..., description="The starting text for the model.")
    max_length: int = Field(default=50, ge=10, le=200, description="Max tokens to generate.")
    temperature: float = Field(default=0.7, gt=0.0, le=2.0, description="Creativity level. Higher is more random.")

# 2. Load the model globally from the local directory
LOCAL_MODEL_DIR = "./model_weights"

try:
    generator = pipeline(
        "text-generation", 
        model=LOCAL_MODEL_DIR, 
        tokenizer=LOCAL_MODEL_DIR
    )
except Exception as e:
    print(f"Failed to load GPT-2 from {LOCAL_MODEL_DIR}. Error: {e}")
    generator = None

@app.post("/generate")
def generate_text(request: GenerationRequest):
    if generator is None:
        raise HTTPException(status_code=500, detail="Model is not loaded.")
    
    if not request.prompt.strip():
        raise HTTPException(status_code=400, detail="Prompt cannot be empty.")

    try:
        # Run the generation pipeline
        result = generator(
            request.prompt,
            max_length=request.max_length,
            temperature=request.temperature,
            num_return_sequences=1,
            # Prevent the pipeline from printing a warning about setting pad_token_id
            pad_token_id=generator.tokenizer.eos_token_id 
        )
        
        # Extract the generated text
        generated_text = result[0]['generated_text']

        return {
            "prompt": request.prompt,
            "generated_text": generated_text,
            "parameters_used": {
                "max_length": request.max_length,
                "temperature": request.temperature
            }
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": generator is not None}
```

## Managing Dependencies (`requirements.txt`)
Create your `requirements.txt`:

```python
fastapi==0.103.2
uvicorn==0.23.2
pydantic==2.4.2
transformers==4.34.0
```

## The Dockerfile
We will execute our downloader script directly inside the Docker build process.

```python
FROM python:3.10-slim

WORKDIR /app

# 1. Install standard requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 2. Install CPU-only PyTorch
RUN pip install --no-cache-dir torch==2.1.0+cpu --index-url https://download.pytorch.org/whl/cpu

# 3. Bake in the model weights
COPY download_model.py .
RUN python download_model.py

# 4. Copy the API application
COPY app.py .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### The `.dockerignore` file

```python
__pycache__/
*.pyc
.venv/
venv/
model_weights/
```

## Deployment Steps
Generative models are computationally expensive.

Build the Docker Image:

```python
docker build -t your-registry/gpt2-api:v1 .
```

Push to your Container Registry:

```python
docker push your-registry/gpt2-api:v1
```

Deploy on the Platform:

* Visit Crane Cloud and create a project to use the image `your-registry/gpt2-api:v1`

Readiness Probes: Always set the platform's health check to the `/health` endpoint to account for the model loading into memory when the container starts.

## Testing the Endpoint
Send a JSON payload with your prompt. Notice how we are also passing our custom generation parameters:

```python
curl -X POST "https://gpt2-api.ahumain.cranecloud.io/generate" \
  -H "Content-Type: application/json" \
  -d '{
        "prompt": "The future of cloud native deployment is",
        "max_length": 30,
        "temperature": 0.8
      }'
```

### Expected Response

```python
{
  "prompt": "The future of cloud native deployment is",
  "generated_text": "The future of cloud native deployment is highly automated, scalable, and built directly on top of containers like Docker to ensure rapid, seamless delivery.",
  "parameters_used": {
    "max_length": 30,
    "temperature": 0.8
  }
}
```