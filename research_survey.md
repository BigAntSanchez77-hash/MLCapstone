# Step 4: Research & Existing Solutions Survey

Papers and public implementations reviewed for the Music Genre Classifier capstone (GTZAN dataset, 10-genre audio classification).

## Papers

1. **Tzanetakis, G. & Cook, P. (2002). "Musical Genre Classification of Audio Signals."** *IEEE Transactions on Speech and Audio Processing*, 10(5).
   https://www.cs.cmu.edu/~gtzan/work/pubs/tsap02gtzan.pdf
   The paper that introduced the GTZAN dataset. 10 genres, handcrafted timbral/rhythmic/pitch features, Gaussian/GMM/kNN classifiers. Best accuracy: **61%** (human baseline ~70%).

2. **Choi, K., Fazekas, G., Sandler, M., & Cho, K. (2017). "Transfer Learning for Music Classification and Regression Tasks."** ISMIR 2017.
   https://arxiv.org/abs/1703.09179
   CNN pre-trained on Million Song Dataset tagging, features transferred to GTZAN genre classification. **89.8%** accuracy vs. 66.0% for an MFCC-only baseline in the same paper; notes contemporary SOTA above 94.5%.

3. **Chatterjee, S., Ganguly, S., Bose, A., Prasad, H. R., & Ghosal, A. (2024). "Audio Processing using Pattern Recognition for Music Genre Classification."** arXiv:2410.14990.
   https://arxiv.org/pdf/2410.14990
   5-genre subset (blues, classical, jazz, hip-hop, country). ZCR, spectral centroid/rolloff, MFCCs, chroma; StandardScaler. KNN 87%, Logistic Regression 86%, **Random Forest 89%**, ANN **92.44%**.

4. **Yadav, A. (2025). "Music Genre Classification Using Audio Features: A Machine Learning Approach With The GTZAN Dataset."** IOSR Journal of Computer Engineering, 27(3).
   https://www.iosrjournals.org/iosr-jce/papers/Vol27-issue3/Ser-3/B2703030516.pdf
   Full 10-genre GTZAN. MFCC1-13, spectral centroid/bandwidth/rolloff, ZCR, tempo, RMS, spectral contrast. Random Forest **78.2%**, SVM 72.4%, KNN 69.1%. MFCCs = 42.6% of RF feature importance.

## Code repositories

5. **chittalpatel/Music-Genre-Classification-GTZAN** (GitHub)
   https://github.com/chittalpatel/Music-Genre-Classification-GTZAN
   Full 10-genre GTZAN. Compares classical ML (LGBM/SVM/XGBoost on librosa features, ~65% accuracy) against a CNN on mel-spectrogram images (~75% accuracy). Includes runnable notebooks for both pipelines.

## Aggregators / further reading

6. **Papers with Code — Music Genre Classification on GTZAN (leaderboard)**
   https://paperswithcode.com/sota/music-genre-classification-on-gtzan
   Tracks published state-of-the-art results on this exact benchmark over time; useful for spotting newer approaches as the capstone progresses.

---

## Candidate chosen for reproduction

**Chatterjee et al. (2024)** — chosen because its pipeline (StandardScaler → Random
Forest, `n_estimators=1000, max_depth=10, criterion='gini'`) is fully specified and
in the same classical-ML family this capstone already uses, unlike the CNN/transfer-
learning papers which need a different data representation (spectrogram images
instead of the extracted feature CSV this project has).

The reproduction attempt — run on this project's own `data/data.csv`, matching the
paper's 5-genre subset and then extended to the full 10-genre scope — is in
`Survey_of_Existing_Research_and_Available_Solutions.ipynb` in this repo. Summary:
reproduced 79% vs. the paper's reported 89% on the matched 5-genre subset; 66% at
full 10-genre scope, in line with Tzanetakis & Cook (61%) and chittalpatel's
classical-ML result (~65%).

## Discussion points:

- Classical ML (Random Forest on handcrafted features) tops out around 66-78%
  accuracy at 10 genres across every source surveyed; deep learning (CNN on
  spectrograms, or transfer-learned embeddings) reaches 75-90%+ but needs raw
  audio, not just an extracted-feature CSV.
- MFCCs are consistently the dominant signal for genre across all sources.

