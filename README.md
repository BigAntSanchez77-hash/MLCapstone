# 🎵 Music Genre Classifier

A machine learning system that classifies music audio into 10 genres using audio feature extraction and scikit-learn. Deployed as a free web application on Hugging Face Spaces.

**[▶ Try the Live App](https://huggingface.co/spaces/AntSanchez77/music-genre-classifier)** · **[📦 Dataset](https://huggingface.co/datasets/AntSanchez77/gtzan-music-genre-dataset)** · **[📓 API Demo Notebook](demo_api_usage.ipynb)**

---

## Quick Start

### Use the Web App

Go to [huggingface.co/spaces/AntSanchez77/music-genre-classifier](https://huggingface.co/spaces/AntSanchez77/music-genre-classifier), upload an audio file, and get an instant genre prediction.

### Use the API (Python)

```bash
pip install gradio_client
```

```python
from gradio_client import Client

client = Client("AntSanchez77/music-genre-classifier")
result = client.predict(audio="song.wav", api_name="/predict")
print(result)
# {'label': 'rock', 'confidences': [{'label': 'rock', 'confidence': 0.87}, ...]}
```

### Use the API (cURL)

```bash
curl -X POST https://AntSanchez77-music-genre-classifier.hf.space/api/predict \
  -H "Content-Type: multipart/form-data" \
  -F "data=@song.wav"
```

---

## Install & Build from Scratch

### Prerequisites

- Python 3.9+
- pip

### Local Setup

```bash
# Clone the repository
git clone https://github.com/AntSanchez77/music-genre-classifier.git
cd music-genre-classifier

# Install dependencies
pip install -r requirements.txt

# Run the app locally
python app.py
# Opens at http://localhost:7860
```

### Train the Model from Scratch

```bash
# 1. Run Step 7 notebook (model experimentation)
jupyter notebook capstone_step7_model_experimentation.ipynb
# → Produces: best_model_pipeline.joblib, label_encoder.joblib, feature_columns.json

# 2. Run Step 8 notebook (scaling)
jupyter notebook capstone_step8_scale_prototype.ipynb

# 3. Copy artifacts to the app directory and run
python app.py
```

### Deploy to Hugging Face Spaces

```bash
# 1. Create a new Space at huggingface.co/new-space (SDK: Gradio)
# 2. Clone your Space repo
git clone https://huggingface.co/spaces/AntSanchez77/music-genre-classifier
cd music-genre-classifier

# 3. Copy these files into the Space repo:
#    app.py, requirements.txt, README.md,
#    best_model_pipeline.joblib, label_encoder.joblib, feature_columns.json

# 4. Push to deploy
git add .
git commit -m "Deploy genre classifier"
git push
```

---

## Project Structure

```
music-genre-classifier/
├── app.py                                  # Gradio web application
├── requirements.txt                        # Python dependencies
├── README.md                               # This file
├── best_model_pipeline.joblib              # Trained model (StandardScaler + classifier)
├── label_encoder.joblib                    # Genre label encoder
├── feature_columns.json                    # Feature column order
├── capstone_step7_model_experimentation.ipynb   # Model comparison & tuning
├── capstone_step8_scale_prototype.ipynb         # Scaling pipeline
├── capstone_step9_deployment_plan.md            # Deployment method selection
├── capstone_step10_deployment_architecture.md   # Architecture design
├── demo_api_usage.ipynb                         # API usage demo notebook
└── genre_classifier.py                          # Standalone classifier module
```

---

## How It Works

### Feature Extraction

The app extracts ~80 audio features from each track using librosa:

| Feature Group | Count | Description |
|---|---|---|
| MFCCs | 40 | 20 mel-frequency cepstral coefficients (mean + std) |
| Delta MFCCs | 20 | First derivative of MFCCs (dynamics) |
| Spectral | 16 | Centroid, bandwidth, rolloff, contrast, flatness |
| Chroma | 24 | 12 pitch class profiles (mean + std) |
| Rhythmic | 3 | Tempo, zero-crossing rate |
| Energy | 2 | RMS energy (mean + std) |
| Harmonic | 6 | Tonnetz tonal centroids |

### Model Pipeline

1. **StandardScaler** normalizes all features to zero mean and unit variance
2. **Classifier** (best model selected from 10 candidates in Step 7) predicts the genre
3. Probability distribution across all 10 genres is returned

### Supported Genres

Blues · Classical · Country · Disco · Hip-Hop · Jazz · Metal · Pop · Reggae · Rock

### Training Data

[GTZAN Music Genre Dataset](https://huggingface.co/datasets/AntSanchez77/gtzan-music-genre-dataset) — 860 audio tracks, 30 seconds each, 10 genres.

---

## Capstone Steps

| Step | Description | Deliverable |
|---|---|---|
| 7 | Experiment with various models | [Notebook](Capstone_ModelExperiment.ipynb) — 10 models compared, hyperparameter tuning, ensembles |
| 8 | Scale your prototype | [Notebook](capstone_step8_scale_prototype.ipynb) — parallel extraction, PySpark, batch inference |
| 9 | Pick deployment method | [Plan](capstone_step9_deployment_plan.md) — Gradio on HF Spaces selected |
| 10 | Design deployment architecture | [Architecture](capstone_step10_deployment_architecture.md) — diagram, write-up, checklist |
| 11 | Deploy to production | This repository + [Live App](https://huggingface.co/spaces/AntSanchez77/music-genre-classifier) |

---

## License

MIT
