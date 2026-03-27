# Deploying Industrial Computer Vision: Tyre Defect Detection

In this guide, we will deploy a computer vision model designed for manufacturing quality assurance. Specifically, we are deploying a **MobileNet** model trained to inspect images of tyres on an assembly line and flag structural defects.

**Why MobileNet ?** 

In industrial settings, cameras capture hundreds of images per minute. Heavy models like ResNet or VGG often require expensive GPUs to keep up with this throughput. 

MobileNet is a highly optimized, lightweight architecture that can run incredibly fast on standard CPUs, drastically reducing your deployment costs while maintaining high accuracy.

---

## Prerequisites

Before starting, ensure you have:

* A trained MobileNet model file (e.g., `tyre_mobilenet_v2.h5`).
* Docker installed locally.

---

## The Inference API (`app.py`)

We will use **FastAPI** to handle image uploads. Since we are using MobileNet, we need to ensure the incoming image is preprocessed exactly how the model expects (typically 224x224 resolution and scaled using MobileNet's specific preprocessing function).

Create a file named `app.py`:

```python
import io
import numpy as np
from PIL import Image
from fastapi import FastAPI, UploadFile, File, HTTPException
import tensorflow as tf
from tensorflow.keras.applications.mobilenet_v2 import preprocess_input

app = FastAPI(title="Industrial Defect Detection API")

# 1. Load the lightweight model globally
try:
    # Replace with your actual model filename
    model = tf.keras.models.load_model('tyre_mobilenet_v2.h5')
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

# 2. Define your manufacturing classes
CLASS_NAMES = ["Defect_Free", "Sidewall_Crack", "Tread_Wear", "Puncture"]

def prepare_image(image_bytes: bytes):
    """Resizes and applies MobileNet-specific preprocessing."""
    img = Image.open(io.BytesIO(image_bytes)).convert("RGB")
    # MobileNetV2 typically expects 224x224
    img = img.resize((224, 224))
    
    img_array = np.array(img)
    img_array = np.expand_dims(img_array, axis=0)
    
    # Crucial: Apply the exact preprocessing used during training
    processed_image = preprocess_input(img_array)
    return processed_image

@app.post("/inspect")
async def inspect_tyre(file: UploadFile = File(...)):
    if model is None:
        raise HTTPException(status_code=500, detail="Model failed to load.")
    
    if not file.content_type.startswith("image/"):
        raise HTTPException(status_code=400, detail="File provided is not an image.")

    try:
        # Read and prepare the image
        contents = await file.read()
        processed_image = prepare_image(contents)

        # Run fast CPU inference
        predictions = model.predict(processed_image)
        predicted_class_idx = np.argmax(predictions[0])
        confidence = float(predictions[0][predicted_class_idx])
        
        diagnosis = CLASS_NAMES[predicted_class_idx]

        return {
            "filename": file.filename,
            "inspection_result": diagnosis,
            "confidence": round(confidence, 4),
            "action": "PASS" if diagnosis == "Defect_Free" else "REJECT_TO_MANUAL_REVIEW"
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    return {"status": "healthy", "model_loaded": model is not None}
```

## Managing Dependencies (`requirements.txt`)
Because MobileNet runs so efficiently, we absolutely want to use the CPU-only version of TensorFlow to keep our Docker image size small.

Create your `requirements.txt`:

```python
fastapi==0.103.2
uvicorn==0.23.2
python-multipart==0.0.6
Pillow==10.0.1
tensorflow-cpu==2.14.0
```

## The Dockerfile
This Dockerfile emphasizes a lean build for fast deployments.

```python
FROM python:3.10-slim

WORKDIR /app

# Install system dependencies required by OpenCV/Pillow if needed
RUN apt-get update && apt-get install -y \
    libgl1-mesa-glx \
    libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the API code and the .h5 model file
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
```

## Deployment Steps
Deploying lightweight vision models gives you great flexibility on our platform.

Build the Docker Image:

```python  
docker build -t your-registry/tyre-defect-api:v1 .
```

Push to your Container Registry:

```python  
docker push your-registry/tyre-defect-api:v1
```

Deploy on the Platform:

* Visit Crane Cloud and create a project to deploy the image `your-registry/tyre-defect-api:v1` 

* Auto-Scaling: Assembly lines can process items in rapid bursts. Crane Cloud handles horizontal auto-scaling for you based on CPU utilization to handle sudden spikes in image uploads from the factory floor cameras.

## Testing the Endpoint
Send a test image from your local machine to simulate a factory camera taking a photo of a tyre:

```python
curl -X POST "https://tyre-defect-api.ahumain.cranecloud.io/inspect" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@path/to/local/tyre_scan_001.jpg"
```

### Expected Response

```python
{
  "filename": "tyre_scan_001.jpg",
  "inspection_result": "Sidewall_Crack",
  "confidence": 0.9812,
  "action": "REJECT_TO_MANUAL_REVIEW"
}
```

