# Machine Learning on Crane Cloud

Training a model is only half the battle. Taking a model from a Jupyter Notebook and turning it into a live, scalable API is a completely different challenge. These guides are designed to help you bridge that gap, teaching you how to wrap your models in lightweight APIs, containerize them efficiently with Docker, and deploy them on our platform.

---

## Which Guide to Choose?

Machine learning deployments vary wildly depending on the type of data and the size of the model. Use this table to find the guide that best matches your project:

| Domain | Example Use Cases | Key Deployment Challenges | Guide to Follow |
| :--- | :--- | :--- | :--- |
| **Healthcare & Vision** | Brain Tumor Aid, Tyre Defect Detection | Image preprocessing (tensors), high RAM usage | [Vision Deployments](healthcare/brain-tumor.md) |
| **NLP & Generative AI** | Hugging Face APIs, GPT-2 Wrappers | Massive model weights, long cold-start times | [GenAI Deployments](natural-language-processing/huggingface-api.md) |
| **Tabular & Predictive** | Fraud Detection, Diabetes Risk Model | High throughput, strict payload validation | [Predictive Deployments](cybersecurity/fraud-detection.md) |

---

## Golden Rules for Machine Learning Containers

Before you dive into a specific deployment guide, review these best practices. Machine learning containers are notoriously difficult to optimize. Following these rules will save you hours of debugging.

### 1. Control Your Image Size
Machine learning libraries like PyTorch and TensorFlow are enormous. If you aren't careful, your Docker image will be several gigabytes, making deployments agonizingly slow.

* **Use slim base images:** Always opt for stripped-down OS base images (e.g., `FROM python:3.10-slim`).
* **Use a `.dockerignore` file:** This is non-negotiable. Never accidentally copy your training datasets (`/data`), local virtual environments (`venv/`), or `.ipynb` notebooks into your production image.

### 2. Bake Your Model Weights into the Image
For models under 1GB-2GB, you should download the model weights `.h5`, `.pt`, or `.bin` files *during* the Docker build process. 

> **Important Note:** Do not configure your API to download the model from the internet every time the container starts. This will cause severe delays, consume unnecessary bandwidth, and likely cause our platform's health checks to terminate your container before it finishes loading.

### 3. Strictly Pin Your Dependencies
The Machine Learning ecosystem moves fast, and breaking changes between library versions are common. Always specify exact versions in your `requirements.txt`.

* **Bad:** `torch`, `transformers`, `fastapi`
* **Good:** `torch==2.1.0`, `transformers==4.34.0`, `fastapi==0.103.2`

### 4. Account for `Cold Start` Times
Large models (especially NLP and GenAI models) can take anywhere from 5 to 30 seconds to load into your application's memory when the container first boots up. You must configure **Readiness Probes** on your deployment so our platform knows your API is `warming up` and won't route user traffic to it until the model is fully loaded.

---

## Ready to Deploy?
Select a guide from the navigation menu to get started with your specific architecture!