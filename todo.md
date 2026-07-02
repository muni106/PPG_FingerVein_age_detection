# Project TODO — Age estimation from FingerVein + PPG
---

## 0. Standing blocker — official data

- [ ] **(CHECKPOINT)** The proposal names *non-public* FingerVein and PPG
  datasets; we currently only have public archives in `proj_files/`
  (MMCBNU_6000, BUT PPG). Confirm with the professor whether the official data
  is available. Until then, **all results below are EXPLORATORY** and must be
  labelled as such. Do not silently treat the public data as the project data.
- [ ] FingerVein ages come from the MMCBNU_6000 description PDF (notebook 04),
  not an official label file. Every FingerVein-age result stays EXPLORATORY with
  explicit provenance.

## 1. Done — data exploration

- [x] `01_fingervein_exploration.ipynb` — structure, demographics, pixel stats.
- [x] `02_ppg_exploration.ipynb` — subject info, age/gender, raw & filtered PPG.
- [x] `03_combined_dataset.ipynb` — subject-level split + post-split age/sex
  pairing, sanity checks.
- [~] `04_extracts_fingervein_age_labels.ipynb` — parse per-subject age/sex from
  the bundled PDF. Finish and **save a versioned labels CSV** (with provenance
  header) so later notebooks load a stable file instead of re-parsing the PDF.

## 2. Shared infrastructure (common work — build once, reuse everywhere)
**same splits** and the **same metrics** for the comparison to be fair.

- [ ] **Frozen subject-level splits.** Persist the train/val/test subject IDs
  for each modality to disk (e.g. JSON/CSV) so every model notebook loads the
  identical split. Split by **subject**, never by image/segment. Test set stays
  untouched during model selection. *(Lab: DL_01 split section.)*
- [ ] **Metrics module.** One helper computing MAE, RMSE, Pearson's r, R² from
  `(y_true, y_pred)`. Reuse in all notebooks. *(Lab: DL_01 RMSE.)*
- [ ] **Error-analysis helpers.** (a) error-vs-age plot, (b) Bland–Altman plot,
  (c) table of highest-error cases. Write once, reuse for A, B, fusion, C.
- [ ] **Normalization rule.** Decide and document: fit all scalers/transforms on
  **train only**, apply to val/test. Applies to PPG normalization and
  FingerVein pixel scaling. *(Lab: DL_01 normalization; DL_02 image preproc.)*
- [ ] Decide where shared code lives (small `src/` module vs. a helpers cell).
  Prefer a tiny importable module so notebooks stay readable.

## 3. Preprocessing pipelines

### PPG (1D) — proposal §2
- [ ] Resample to a fixed rate; window into fixed-length segments.
- [ ] Band-pass filter (already prototyped in nb 02) as a reusable function.
- [ ] Per-window normalization (fit on train).
- [ ] Output tensor shape documented, e.g. `(n_windows, window_len, 1)`.
- [ ] **(CHECKPOINT)** target definition: one age per subject → how are
  multiple windows per subject handled at train vs. eval time (aggregate
  predictions per subject?). *(Lab: DL_03 sequence prep.)*

### FingerVein (2D) — proposal §2
- [ ] ROI extraction (or documented decision to skip and use full image first).
- [ ] Illumination/contrast normalization, resize to fixed resolution.
- [ ] Pixel normalization (fit on train).
- [ ] Light augmentation (small rotations/translations) to limit overfitting.
- [ ] Output tensor shape documented, e.g. `(H, W, 1)`. *(Lab: DL_02 image
  preprocessing & augmentation.)*

## 4. Baselines before neural nets (required by AGENTS.md)

- [ ] **PPG baseline:** predict-the-mean-age + a simple regressor on a few
  hand-features (e.g. heart-rate/peak stats). Establishes the floor MAE/RMSE.
- [ ] **FingerVein baseline:** predict-the-mean-age + a simple feature/linear
  baseline. *(Lab: DL_01 regression baseline.)*
- [ ] Record baseline metrics — every deep model must beat these to be worth it.

## 5. Model A — FingerVein (member 1)

- [ ] 2D CNN on vein images → single continuous age output.
  Start from `build_lenet5` (DL_02); optionally transfer learning later.
- [ ] Train with early stopping; plot history (`plot_history`) and predictions
  (`plot_prediction_results`) from the labs.
- [ ] Evaluate on the frozen test set with the shared metrics + error analysis.
- [ ] **(EXPLORATORY)** — labels from PDF.

## 6. Model B — PPG (member 2)

- [ ] 1D-CNN on PPG windows → single continuous age output.
- [ ] Optional recurrent variant (LSTM/GRU) to capture wave morphology.
  Start from `build_simple_rnn`/`build_deep_rnn` (DL_03).
- [ ] Aggregate window-level predictions to a subject-level age for evaluation
  (per the §3 checkpoint decision).
- [ ] Train (early stopping), evaluate on frozen test set + error analysis.

## 7. Model C — joint two-branch (member 3)

- [ ] Two-branch net: 2D-CNN branch (FingerVein) + 1D branch (PPG), embeddings
  concatenated before the regression head = **early / feature-level fusion**.
- [ ] Train on the **combined (paired) dataset** built in notebook 03, using the
  pairing produced *after* the subject split (no leakage).
- [ ] Evaluate on the combined test set + error analysis.

## 8. Score-level fusion (member 3)

- [ ] Combine Model A and Model B outputs: mean / weighted mean, with the
  weight/rule tuned **on validation only**.
- [ ] Evaluate the fused predictor on the test set + error analysis.

## 9. Main comparison & write-up

- [ ] Single comparison table on the **same test split, same metrics**:
  **A alone · B alone · fusion(A,B) · C**. Answers (i) does multimodality beat
  single modalities, and (ii) score-level fusion vs. joint model.
- [ ] Full error analysis for the final experiments: error-by-age (young/old
  bias), Bland–Altman, high-error examples. *(Proposal §4.)*
- [ ] Reflection: limitations, dataset caveats (small/biased, EXPLORATORY
  labels), what is *supported* vs. *exploratory*. Don't overstate.
- [ ] Document seeds; note reproducibility is not perfectly guaranteed.

## 10. Housekeeping

- [ ] Update `README.md` notebook table as new notebooks land (03/04 missing).
- [ ] Keep raw archives, models, and large outputs out of git (`.gitignore`).
- [ ] Each substantive notebook follows the 7-part notebook standard in
  `AGENTS.md` (question → data → method → impl → checks → results → reflection).

---

### Suggested next action

Finish **§1 nb04** (save the labels CSV), then build **§2 shared infra**
(frozen splits + metrics + error-analysis helpers). Everything else depends on
those two being stable.
