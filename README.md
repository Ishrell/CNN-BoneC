# Bone Cancer CNN – Web UI

A minimal full‑stack app to run your Kaggle bone cancer model locally with a clean web UI, dark mode, and optional Grad‑CAM.

https://github.com/ (optional to push)

## Folder Structure

```
bone-cancer-webui/
├─ backend/
│  ├─ main.py
│  └─ requirements.txt
└─ frontend/
   ├─ index.html
   └─ script.js
```

## Prereqs

- Python 3.10+
- (Recommended) Create & activate a virtualenv
- Your trained Keras/TensorFlow model file exported from Kaggle (e.g., `model.h5` or a SavedModel directory)

## Setup – Backend

```bash
cd backend
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
# Put your trained model at ./model.h5  (or set MODEL_PATH env var to point to it)
export MODEL_PATH=./model.h5            # Windows PowerShell: $env:MODEL_PATH="./model.h5"
export IMG_SIZE=224                     # set to whatever the model expects
export CLASS_NAMES="Normal,Malignant"   # comma-separated labels in model's order
uvicorn main:app --reload --port 8000
```

Health check:
- Open http://127.0.0.1:8000/health

## Run – Frontend

Just open `frontend/index.html` in your browser.  
If your API is not at `http://127.0.0.1:8000`, set `window.API_URL_OVERRIDE` before loading `script.js`.

Example:
```html
<script>
  window.API_URL_OVERRIDE = "http://localhost:8000";
</script>
<script src="./script.js"></script>
```

## Notes

- **Preprocessing.** By default, images are resized to `IMG_SIZE` and normalized to `[0,1]`. If your model was trained on grayscale X‑rays, open `backend/main.py` and uncomment the two lines in `preprocess()` to force grayscale.
- **Grad‑CAM.** Enabled by default. If your model has no conv layers (e.g., pure Dense), set `USE_GRADCAM=0`. If Keras can’t auto‑find the last conv layer, set `LAST_CONV_LAYER=your_layer_name`.
- **Binary outputs.** If your model returns a single logit/probability, the API maps it to `[Negative, Positive]` unless you override `CLASS_NAMES` with two labels.
- **Security.** This is a local dev app. Before deploying, add proper input validation, auth, and HTTPS; pin TensorFlow to your environment (CPU/GPU) and consider model quantization.
- **Framework.** The backend uses TensorFlow/Keras. If your Kaggle notebook used PyTorch instead, ping me and I’ll swap in a Torch loader (easy change).

## API

- `POST /predict` (multipart form with `file`): returns `{ class_names, probs, predicted_class, gradcam_image_b64? }`
- `GET /health`

## Troubleshooting

- *Model not found*: set `MODEL_PATH` to your `.h5` or SavedModel dir.
- *Wrong input size*: set `IMG_SIZE` to match your training.
- *Wrong class names/order*: set `CLASS_NAMES` (comma‑separated) to match your training labels.
- *Grad‑CAM shows nothing*: try setting `LAST_CONV_LAYER` to the last convolution layer’s name.
```

