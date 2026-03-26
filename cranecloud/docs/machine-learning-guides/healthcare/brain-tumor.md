# Deploying a Computer Vision Model: Brain Tumor Classifier

Computer vision deployments often involve handling image files, preprocessing them into tensors (arrays of numbers), and passing them through deep learning frameworks like TensorFlow or PyTorch. 

This guide uses **FastAPI** to handle the image uploads and **TensorFlow** for inference.

---

## Prerequisites

Before you begin, ensure you have the following in your project directory:

* A trained model weights file (e.g., `brain_tumor_model.h5`).
* Docker installed on your local machine.

---

## The Inference API (`app.py`)

We need a web server to accept incoming MRI images, resize them to the format our model expects, and return the prediction. FastAPI is perfect for this because it handles `multipart/form-data` (file uploads) quickly and efficiently.

Create a file named `app.py`:

```python
import io
import numpy as np
from PIL import Image
from fastapi import FastAPI, UploadFile, File, HTTPException
import tensorflow as tf

app = FastAPI(title="Brain Tumor Classifier API")

# 1. Load the model globally so it only loads once when the container starts
# Replace 'brain_tumor_model.h5' with your actual model file
try:
    model = tf.keras.models.load_model('brain_tumor_model.h5')
except Exception as e:
    print(f"Error loading model: {e}")
    model = None

# 2. Define labels based on your model's training
CLASS_NAMES = ["No Tumor", "Pituitary Tumor", "Meningioma Tumor", "Glioma Tumor"]

def prepare_image(image_bytes: bytes):
    """Resizes and normalizes the image for the model."""
    img = Image.open(io.BytesIO(image_bytes)).convert("RGB")
    img = img.resize((224, 224)) # Adjust to your model's input size
    img_array = np.array(img) / 255.0
    img_array = np.expand_dims(img_array, axis=0)
    return img_array

@app.post("/predict")
async def predict(file: UploadFile = File(...)):
    if model is None:
        raise HTTPException(status_code=500, detail="Model failed to load.")
    
    if not file.content_type.startswith("image/"):
        raise HTTPException(status_code=400, detail="File provided is not an image.")

    try:
        # Read and prepare the image
        contents = await file.read()
        processed_image = prepare_image(contents)

        # Run inference
        predictions = model.predict(processed_image)
        predicted_class_idx = np.argmax(predictions[0])
        confidence = float(predictions[0][predicted_class_idx])

        return {
            "filename": file.filename,
            "diagnosis": CLASS_NAMES[predicted_class_idx],
            "confidence": confidence
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
def health_check():
    """Used by the platform to check if the container is ready."""
    return {"status": "healthy", "model_loaded": model is not None}
```

## Managing Dependencies (`requirements.txt`)

Create a `requirements.txt` file. **Crucially, specify your library versions.**

```text
fastapi==0.103.2
uvicorn==0.23.2
python-multipart==0.0.6
Pillow==10.0.1
tensorflow-cpu==2.14.0  # Use CPU version to keep image size smaller if no GPU is needed
```

> **Info**: If your deployment doesn't strictly require a GPU for inference, using tensorflow-cpu drastically reduces your Docker image size compared to the standard tensorflow package.

## The Dockerfile

Machine learning containers can get large. We will use a slim Python image to save space. Create a file named `Dockerfile` (no extension):

```python
# Use a lightweight base image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Install system dependencies required by OpenCV/Pillow if needed
RUN apt-get update && apt-get install -y \
    libgl1-mesa-glx \
    libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install them
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application, including the model file
COPY . .

# Expose the port FastAPI runs on
EXPOSE 8000

# Command to run the application
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### The `.dockerignore` file

Create a `.dockerignore` file to prevent copying massive training datasets or local environments into your container:

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
Once your files are ready, it's time to build and deploy to the cluster.

Build the Docker Image:

```python
docker build -t your-registry/brain-tumor-api:v1 .
```

Push to your Container Registry:

```
docker push your-registry/brain-tumor-api:v1
```

Deploy on the Platform: 

* Visit Crane Cloud and create a project to deploy the image `your-registry/brain-tumor-api:v1` 
* Ensure you configure the Health Check / Readiness Probe to point to the `/health` endpoint so the platform knows exactly when your model has finished loading into memory.

## Testing the Endpoint

Once deployed, you can test your live endpoint using `curl` or `Postman`. Pass an actual MRI scan image from your local machine to the API:

```python
curl -X POST "https://brain-tumor-api.ahumain.cranecloud.io/predict" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@path/to/local/mri_scan.jpg"
```

### Expected Response

```python
{
  "filename": "mri_scan.jpg",
  "diagnosis": "Glioma Tumor",
  "confidence": 0.984
}
```