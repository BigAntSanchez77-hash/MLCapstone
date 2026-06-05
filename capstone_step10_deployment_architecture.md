# Capstone Step 10: Deployment Solution Architecture

## Music Genre Classification — Gradio on Hugging Face Spaces

---

## 1. Architecture Diagram

*(See accompanying architecture diagram: `deployment_architecture.png`)*

The architecture diagram is also rendered inline in the project notebook. A simplified summary of the data flow:

```
User uploads .wav → Gradio Interface → librosa Feature Extraction (~80 features)
    → StandardScaler + Classifier (.joblib) → Genre + Probabilities → Display to User
```

All components run within a single Hugging Face Space container (CPU free tier). The retraining pipeline runs offline via the Step 7 Jupyter notebook, producing updated `.joblib` artifacts that are pushed to the Space repository.

---

## 2. Design Decisions Write-Up

### 2.1 System Components

**Inputs:** A `.wav` audio file (up to 30 seconds, any sample rate — resampled to 22050 Hz internally).

**Outputs:** Predicted genre label (one of 10 GTZAN genres) plus a probability distribution bar chart showing confidence across all genres.

**Major components:**

The system consists of four components packaged inside a single container. The Gradio interface handles both the web UI for browser-based users and an auto-generated REST API for programmatic access. When audio arrives, it is passed to the feature extraction module, which uses librosa to compute approximately 80 features (MFCCs, spectral descriptors, chroma, tempo, tonnetz). These features are fed into the trained scikit-learn pipeline (StandardScaler followed by the best classifier from Step 7), which outputs a genre prediction and class probabilities. Finally, the results are rendered back through Gradio as a label plus a bar chart.

### 2.2 Data Storage

Audio files are not stored — they are processed in memory during inference and discarded. The only persistent data is the model artifacts (three files totaling under 5 MB), which live inside the Hugging Face Space's Git repository. The training dataset remains on Hugging Face Datasets at `AntSanchez77/gtzan-music-genre-dataset` and is only accessed during retraining, not during inference.

Extracted features from the training set are cached as a Parquet file during training (Step 8) but are not deployed to production — they are only used offline.

### 2.3 Data Flow Between Components

All data flow happens in-process (no network calls between components). When a user uploads audio via the Gradio UI or sends a POST request to the API endpoint, the audio bytes are passed directly to librosa for feature extraction in Python. The resulting feature vector (a NumPy array) is passed to the scikit-learn pipeline's `.predict()` and `.predict_proba()` methods. The results are returned synchronously to Gradio, which renders the output. The entire request-response cycle takes approximately 1–2 seconds, with feature extraction accounting for roughly 90% of that time.

### 2.4 ML Model Lifecycle

**Retraining frequency:** Retraining is triggered manually when one of two conditions is met: (a) the training dataset grows significantly (e.g., 500+ new labeled tracks are added), or (b) monitoring reveals that prediction accuracy has degraded below an acceptable threshold (e.g., below 70% on a held-out validation set). For a PoC of this scope, a fixed quarterly review cadence is sufficient.

**Retraining data:** New labeled audio tracks are added to the Hugging Face Dataset. The Step 7 notebook is re-run with the expanded dataset, which automatically re-extracts features, re-trains all candidate models, and selects the best performer via cross-validation.

**Validation before deployment:** A retrained model must meet or exceed the current production model's F1 Macro score on a held-out test set (20% stratified split). The Step 7 notebook produces this comparison automatically. If the new model underperforms, it is not deployed.

**Deployment of retrained model:** The updated `.joblib` files are committed to the Hugging Face Space's Git repository. Hugging Face automatically rebuilds and redeploys the Space within 2–3 minutes. No downtime — the old container serves requests until the new one is ready (blue-green deployment is handled by the platform).

**Model artifact storage:** All model versions are tracked in the Space's Git history. Previous versions can be restored via `git revert` if a regression is detected.

### 2.5 Monitoring and Debugging

**Monitoring:** Hugging Face Spaces provides built-in logging (stdout/stderr). The Gradio app logs each prediction (genre, confidence, processing time) to stdout, which is visible in the Space's "Logs" tab. For a PoC, this is sufficient. In a production system, these logs would be shipped to a centralized logging service (e.g., CloudWatch, Datadog) and aggregated into dashboards tracking prediction distribution, confidence trends, and latency.

**Debugging:** If errors occur, the Space logs capture full Python tracebacks. Common failure modes include corrupted audio files (handled by a try/except returning a user-friendly error message) and out-of-memory errors on very long audio files (mitigated by truncating input to the first 30 seconds).

### 2.6 Error Handling and Outages

The Gradio app wraps all inference logic in a try/except block. If feature extraction or prediction fails, the user receives a clear error message rather than a crash. If the Space itself goes down (HF infrastructure outage), Hugging Face automatically restarts it. The free tier has no SLA, but historically HF Spaces availability exceeds 99%. For a production deployment requiring higher reliability, the app could be containerized (Docker) and deployed on AWS/GCP with auto-scaling and health checks.

### 2.7 Technology Choices

| Component | Technology | Rationale |
|---|---|---|
| ML Framework | scikit-learn | Lightweight, no GPU needed, fast inference, proven in Steps 7–8 |
| Audio Processing | librosa | Industry standard, comprehensive feature set, pure Python |
| Web Framework | Gradio | Minimal code for UI + API, native HF integration, free hosting |
| Hosting | Hugging Face Spaces | Free CPU tier, Git-based deployment, auto-scaling, zero DevOps |
| Dataset Storage | Hugging Face Datasets | Already in use, versioned, accessible via `load_dataset()` |
| Model Serialization | joblib | Standard for scikit-learn pipelines, compact file size |
| Version Control | Git (via HF Space repo) | Built-in model versioning and rollback |

---

## 3. Cost Estimate

| Resource | Cost | Notes |
|---|---|---|
| Hugging Face Space (CPU Basic) | **$0/month** | Free tier: 2 vCPU, 16 GB RAM, sufficient for this workload |
| Hugging Face Dataset hosting | **$0/month** | Free for public datasets up to 50 GB |
| Domain/SSL | **$0** | Provided by HF (*.hf.space subdomain) |
| Development time | **~6–8 hours** | Building Gradio app, testing, deploying |
| Retraining compute | **$0** | Runs locally or on Google Colab (free tier) |
| **Total monthly operating cost** | **$0/month** | |

**Scaling costs (if needed in future):**

| Upgrade | Cost | When Needed |
|---|---|---|
| HF Space CPU Upgrade (8 vCPU) | ~$0.07/hour | >100 concurrent users |
| HF Space GPU (T4) | ~$0.60/hour | If switching to deep learning model |
| AWS/GCP dedicated instance | ~$30–100/month | Production SLA requirement |
| Custom domain | ~$12/year | Branding requirement |

---

## 4. Pre-Deployment Checklist

### Functionality
- [ ] Gradio app loads without errors on a clean HF Space
- [ ] Model artifacts (`.joblib`, `.json`) are present and loadable
- [ ] Audio upload accepts `.wav`, `.mp3`, `.ogg`, `.flac` formats
- [ ] Feature extraction completes within 5 seconds for a 30-second clip
- [ ] Predictions return a valid genre label for all 10 classes
- [ ] Probability distribution sums to ~1.0 across all genres
- [ ] API endpoint (`/api/predict`) returns valid JSON responses

### Robustness
- [ ] Corrupted audio files return a user-friendly error, not a crash
- [ ] Very short audio (<1 second) is handled gracefully
- [ ] Very long audio (>5 minutes) is truncated to 30 seconds with a warning
- [ ] Silence-only audio returns a low-confidence prediction (not an error)
- [ ] Non-audio file uploads are rejected with a clear message
- [ ] Concurrent requests (2–3 simultaneous users) don't cause failures

### Performance
- [ ] End-to-end latency is under 5 seconds for a typical 30-second track
- [ ] Memory usage stays under 2 GB during inference
- [ ] Space cold-start time is under 60 seconds

### Model Quality
- [ ] Deployed model matches the best model from Step 7 (same `.joblib`)
- [ ] Test predictions on known tracks match expected genres
- [ ] F1 Macro score on held-out test set meets the Step 7 baseline
- [ ] Confusion matrix shows no single genre with <50% recall

### Documentation
- [ ] Space README describes the project, dataset, and model
- [ ] API usage examples are documented (curl + Python)
- [ ] GitHub repository links to the live Space
- [ ] Model card lists training data, features, and performance metrics

### Security and Privacy
- [ ] No user audio is stored or logged beyond the session
- [ ] No API keys or secrets are exposed in the Space code
- [ ] `requirements.txt` pins dependency versions to avoid supply chain issues

### Rollback Plan
- [ ] Previous model version is tagged in Git history
- [ ] Rollback procedure documented: `git revert` + push to trigger rebuild
- [ ] Tested that rollback produces a working Space within 5 minutes

---

*Prepared for ML Engineering & AI Bootcamp Capstone — Step 10*
