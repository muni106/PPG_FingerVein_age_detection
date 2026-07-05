# Project TODO — Age estimation from FingerVein + PPG
---

## Guiding principle

**Default = replicate the lab.** For every step, reuse the closest professor
example/utility and change as little as possible. **Deviate only where the
proposal forces it**; each deviation is marked **⚠ DEVIATION** with the reason
and the lab it departs from. Follow `proposta-rivista.md` for *what* to do and
`labs/` for *how* to do it.

Lab utilities available to reuse (verified):
- **DL_01**: `build_linear_nn`, `build_dnn`, `plot_history`,
  `plot_prediction_results`, `train_test_split`, `StandardScaler`,
  `root_mean_squared_error`, `EarlyStopping`, `LinearRegression`.
- **DL_02**: `build_lenet5`, `build_alexnet`, `Conv2D`, `BatchNormalization`,
  `Dropout`.
- **DL_03**: `build_simple_rnn`, `build_deep_rnn`, `SimpleRNN` (sequence prep).
- **DL_07**: `build_1d_cnn` (1D conv template — classification, adapt to regression).

Metrics reality check: **labs implement only RMSE** (`root_mean_squared_error`).
MAE, Pearson's r, and R² are **⚠ DEVIATION (proposal §4)** — not in any lab; add
via `sklearn.metrics.mean_absolute_error`, `r2_score`, `scipy.stats.pearsonr`.

## 0. Standing blocker — official data

- [ ] **(CHECKPOINT)** The proposal (§2) says *non-public* datasets and
  explicitly **not public ones**; we only have public archives in `proj_files/`
  (MMCBNU_6000, BUT PPG). Confirm with the professor. Until then **all results
  are EXPLORATORY** and labelled as such.
- [ ] FingerVein ages come from the MMCBNU_6000 PDF (nb04), not an official label
  file. Every FingerVein-age result stays EXPLORATORY with explicit provenance.

## 1. Done — data exploration

- [x] `01_fingervein_exploration.ipynb` — structure, demographics, pixel stats,
  + frozen FingerVein split.
- [x] `02_ppg_exploration.ipynb` — subject info, age/gender, raw & filtered PPG,
  + frozen PPG split.
- [x] `03_combined_dataset.ipynb` — loads the two frozen splits, post-split
  age/sex pairing, sanity checks.
- [x] `04_extracts_fingervein_age_labels.ipynb` — parse per-subject age/sex from
  the PDF → versioned `notebooks/subject_age_labels.csv` (provenance header).

## 2. Shared infrastructure (common work — build once, reuse everywhere)

Everything below feeds the fair comparison: **same splits, same metrics** for all
models (proposal §3, "valutati sugli stessi split e con le stesse metriche").

- [x] **Frozen subject-level splits.** Lab basis: DL_01 `train_test_split`
  (70/15/15, `random_state=0`), per subject. nb01 → `fingervein_splits.csv`,
  nb02 → `ppg_splits.csv`; nb03 loads both. Note: combined val/test end up tiny
  (val 2 FV/2 PPG, test 8 FV/4 PPG ids) — flag when reporting Model C.
- [ ] **Metrics helper.** `metrics(y_true, y_pred)` → MAE, RMSE, r, R².
  Lab basis: DL_01 `root_mean_squared_error` for RMSE.
  **⚠ DEVIATION (proposal §4):** MAE/r/R² absent from labs — add
  `mean_absolute_error`, `r2_score`, `scipy.stats.pearsonr`. One line each,
  documented.
- [ ] **Prediction/error plots.** Reuse DL_01 `plot_prediction_results`
  (true-vs-pred + error distribution) unchanged for A, B, fusion, C.
- [ ] **Error-analysis helpers.** (a) error-vs-age plot, (b) Bland–Altman plot,
  (c) highest-error cases table.
  **⚠ DEVIATION (proposal §4):** error-by-age and Bland–Altman are not in the
  labs — small custom helpers (matplotlib only). "Best/worst predictions" pattern
  is reusable from DL_01 for (c).
- [ ] **Normalization rule.** Fit on **train only**, apply to val/test
  (DL_01/DL_03: `StandardScaler().fit(train)` then transform). PPG → StandardScaler
  per DL_03; FingerVein pixels → `/255` scaling per DL_02.
- [ ] **Where shared code lives.** Follow the labs: define helpers in a **cell at
  the top of each model notebook** (as DL_01 defines `plot_history` etc.), *not* a
  separate `src/` package. Copy the small metrics/plot helpers into each notebook
  so notebooks stay self-contained like the labs.

## 3. Preprocessing pipelines (proposal §2)

### PPG (1D)
- [ ] Resample to a fixed rate; window into fixed-length segments. Lab basis:
  DL_03 sequence preparation. **⚠ DEVIATION:** signal resampling itself isn't in
  DL_03 — use `scipy.signal` if resampling is needed; keep it minimal.
- [ ] Band-pass filter (already prototyped in nb02) as a reusable function.
  **⚠ DEVIATION (proposal §2):** domain signal filtering, no lab equivalent.
- [ ] Per-window normalization, fit on train (DL_03 `StandardScaler`).
- [ ] Document output shape, e.g. `(n_windows, window_len, 1)` (DL_03 style).
- [ ] **(CHECKPOINT)** target definition: one age per subject → how are multiple
  windows per subject aggregated at eval time? *(Lab: DL_03 sequence prep.)*

### FingerVein (2D)
- [ ] ROI extraction — **already provided**: MMCBNU_6000 ships `ROIs/`, so use
  them directly (documented decision to skip custom extraction for now).
- [ ] Illumination/contrast normalization + resize to fixed resolution
  (DL_02 image preprocessing).
- [ ] Pixel normalization `/255`, consistent across splits (DL_02).
- [ ] Light augmentation (small rotations/translations). **⚠ DEVIATION
  (proposal §2, optional):** only if overfitting appears; check DL_02 for the
  augmentation technique before adding.
- [ ] Document output shape, e.g. `(H, W, 1)` (DL_02).

## 4. Baselines before neural nets (required by AGENTS.md)

- [ ] **PPG baseline:** predict-the-mean-age, then simple regressor on a few
  hand-features. Lab basis: DL_01 `LinearRegression` + RMSE. Establishes floor.
- [ ] **FingerVein baseline:** predict-the-mean-age + simple linear baseline
  (DL_01).
- [ ] Record baseline metrics — every deep model must beat these.

## 5. Model A — FingerVein (member 1)

- [ ] 2D CNN → single continuous age output. Lab basis: `build_lenet5` (DL_02),
  change the head to **1 linear output** + `loss='mse'`.
- [ ] Train with `EarlyStopping`; `plot_history` + `plot_prediction_results` (DL_01/02).
- [ ] Evaluate on the frozen FingerVein test set with the §2 metrics helper +
  error analysis.
- [ ] Optional transfer learning (proposal §3). **⚠ DEVIATION:** not in labs —
  defer behind a research checkpoint. **(EXPLORATORY — labels from PDF.)**

## 6. Model B — PPG (member 2)

- [ ] 1D-CNN → single continuous age output. Lab basis: `build_1d_cnn` (DL_07),
  adapt from classification to **1 regression output** + `loss='mse'`.
- [ ] Optional recurrent variant: `build_simple_rnn` / `build_deep_rnn` (DL_03).
  LSTM/GRU is **⚠ DEVIATION (proposal §3, optional)** — DL_03 uses `SimpleRNN`
  only; add LSTM/GRU only if the simple RNN underperforms.
- [ ] Aggregate window-level predictions to a subject-level age for eval
  (per §3 checkpoint).
- [ ] Train (`EarlyStopping`), evaluate on frozen PPG test set + error analysis.

## 7. Model C — joint two-branch (member 3)

- [ ] Two-branch net: CNN-2D branch (reuse Model A body) + 1D branch (reuse Model
  B body), embeddings concatenated before the regression head = **early/feature-
  level fusion** (proposal §3).
  **⚠ DEVIATION:** multi-input functional model isn't in the labs — build with
  the Keras functional API, reusing each branch from the single-modality models.
- [ ] Train on the combined dataset from nb03 (pairing done *after* the split).
- [ ] Evaluate on the combined test set + error analysis; report with the tiny-
  identity caveat from nb03.

## 8. Score-level fusion (member 3)

- [ ] Combine Model A and Model B outputs (mean / weighted mean), weight tuned
  **on validation only** (proposal §3). **⚠ DEVIATION:** no lab equivalent —
  simple numpy combination of the two models' predictions.
- [ ] Evaluate the fused predictor on the test set + error analysis.

## 9. Main comparison & write-up (proposal §3–§4)

- [ ] Single comparison table on the **same test split, same metrics**:
  **A · B · fusion(A,B) · C** → answers (i) multimodality vs single, (ii)
  score-fusion vs joint model.
- [ ] Full error analysis: error-by-age (young/old bias), Bland–Altman,
  high-error examples.
- [ ] Reflection: limitations, EXPLORATORY caveats, supported vs exploratory.
- [ ] Document seeds; note reproducibility is not perfectly guaranteed.

## 10. Housekeeping

- [ ] Update `README.md` notebook table (03/04 missing).
- [ ] Keep raw archives, models, large outputs out of git.
- [ ] Each substantive notebook follows the 7-part standard in `AGENTS.md`.

---

### Suggested next action

Build the **§2 metrics helper** (RMSE from DL_01 + the three proposal metrics),
then the **prediction/error-analysis plots**. Both are reused by every model, so
they come before Models A/B/C.
