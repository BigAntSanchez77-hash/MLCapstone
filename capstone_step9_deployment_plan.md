# Capstone Step 9: Deployment Plan — Music Genre Classification

## Deployment Method Selected: Hugging Face Space (Gradio App)

### Why This Approach

After evaluating three deployment options, a **Gradio application deployed as a Hugging Face Space** was selected as the best fit for this project.

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| FastAPI REST API | Standard for production ML; scalable | Requires separate hosting (AWS/GCP); no built-in UI | Good for enterprise, overkill for PoC |
| Streamlit Web App | Easy to build; interactive | Requires hosting; limited API support | Good for dashboards, less suited for single-task ML |
| **Gradio on HF Space** | **Free hosting; built-in API + UI; ties into existing HF dataset; easy to share and demo** | Limited customization; dependent on HF infrastructure | **Best fit for PoC** |

### Key Reasons

- The trained model artifacts (`.joblib` pipeline, label encoder, feature columns) are lightweight and compatible with Gradio's serverless architecture.
- Hugging Face Spaces provides free CPU-based hosting, which is sufficient since our model is a traditional ML pipeline (no GPU needed).
- Gradio automatically generates both a **web UI** (for demos) and a **REST API** (for programmatic access), satisfying multiple use cases with one deployment.
- The dataset already lives on Hugging Face (`AntSanchez77/gtzan-music-genre-dataset`), creating a cohesive project ecosystem.

---

## Deployment Architecture

```
User uploads .wav file
        │
        ▼
┌──────────────────────────────────┐
│   Hugging Face Space (Gradio)    │
│                                  │
│  ┌────────────────────────────┐  │
│  │  1. Audio Input (file)     │  │
│  │  2. librosa feature        │  │
│  │     extraction (~80 feats) │  │
│  │  3. StandardScaler +       │  │
│  │     Model.predict()        │  │
│  │  4. Return genre label     │  │
│  │     + confidence bars      │  │
│  └────────────────────────────┘  │
│                                  │
│  Model: best_model_pipeline.     │
│         joblib (from Step 7)     │
└──────────────────────────────────┘
        │
        ▼
  Genre prediction displayed
  (e.g., "Rock — 87% confidence")
```

---

## Next Steps: Implementation Checklist

### Step 1: Prepare the Gradio Application
- [ ] Create `app.py` with Gradio interface that accepts an audio file upload
- [ ] Integrate the `extract_features()` function from the capstone pipeline
- [ ] Load the trained model pipeline (`best_model_pipeline.joblib`)
- [ ] Display prediction results with genre label and probability distribution across all 10 genres

### Step 2: Create the Hugging Face Space
- [ ] Create a new Space at `huggingface.co/spaces/AntSanchez77/music-genre-classifier`
- [ ] Select **Gradio** as the SDK
- [ ] Choose **CPU Basic** (free tier) — sufficient for sklearn/librosa inference

### Step 3: Upload Required Files
- [ ] `app.py` — Gradio application code
- [ ] `requirements.txt` — Python dependencies (librosa, scikit-learn, xgboost/lightgbm, joblib, gradio)
- [ ] `best_model_pipeline.joblib` — trained model from Step 7
- [ ] `label_encoder.joblib` — genre label encoder from Step 7
- [ ] `feature_columns.json` — feature column order from Step 7

### Step 4: Test and Validate
- [ ] Test with sample audio files from each of the 10 genres
- [ ] Verify the API endpoint works for programmatic access
- [ ] Confirm predictions match Step 7 holdout test results
- [ ] Test edge cases: short clips, silence, non-music audio

### Step 5: Documentation and Sharing
- [ ] Add a model card / README to the Space describing the project
- [ ] Link to the dataset (`AntSanchez77/gtzan-music-genre-dataset`)
- [ ] Link to the GitHub repository with notebooks from Steps 7 and 8
- [ ] Share the public Space URL with mentor for review

---

## Engineering Considerations

### Performance
- Single-track inference takes ~1–2 seconds (feature extraction is the bottleneck, not prediction).
- HF Spaces free tier provides 2 vCPUs and 16 GB RAM — more than enough for this pipeline.
- For higher throughput, the Space can be upgraded to a paid tier or the model can be moved behind a dedicated FastAPI service.

### Limitations of Current Approach
- Audio must be uploaded as a file (no real-time streaming).
- Free-tier Spaces spin down after inactivity; first request after idle has ~30s cold start.
- Limited to the 10 GTZAN genres; new genres require retraining.

### Future Enhancements
- Add a deep learning model (CNN on mel spectrograms) as an alternative prediction path for higher accuracy.
- Support real-time audio recording via the Gradio microphone input.
- Expand genre coverage beyond the 10 GTZAN categories.
- Add batch upload support for classifying multiple tracks at once.
- Integrate with a music recommendation engine or media library organizer.

---

## Technology Stack Summary

| Component | Technology | Reason |
|---|---|---|
| Model | scikit-learn pipeline (StandardScaler + best classifier from Step 7) | Lightweight, fast inference, no GPU needed |
| Feature Extraction | librosa | Industry standard for audio feature extraction |
| Web UI + API | Gradio | Provides both UI and API with minimal code |
| Hosting | Hugging Face Spaces (free CPU tier) | Free, reliable, integrates with HF ecosystem |
| Dataset | Hugging Face Datasets | Already hosted at `AntSanchez77/gtzan-music-genre-dataset` |
| Version Control | GitHub | Notebooks, model code, and deployment config |

---

*Prepared for ML Engineering & AI Bootcamp Capstone — Step 9*
