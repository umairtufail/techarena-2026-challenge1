# Understand.md — TechArena 2026, Challenge 1: Battery Lifetime Prediction

Status: v1, written after reading `README.md`, `submission_instructions_2026.md`,
`TechArena_2026_Topic_1_Challenge1.pdf` and every file under `framework/`,
`my_model/`, plus a direct inspection of the real dataset.

---

## 1. The one-sentence task

Build a model that, given measured data from a handful of aged 102 Ah LFP
prismatic cells (the fine-tuning set), predicts the full SOH%-vs-cycle curve
(cycle 1 down to 70% SOH, including the knee) for **any** operating point in
25–55 °C × 0.5–1.0 C — including (a) sibling cells of the released conditions
and (b) operating points we never saw during training.

## 2. What the organizers give us

| Item | Where | Purpose |
|---|---|---|
| Fixed framework | `framework/`, `run_model.py`, `validate_submission.py` | Frozen evaluation harness — **never edit** |
| Model skeleton | `my_model/model_template.py` | Our file: implement `fit()` + `predict_soh()` |
| Reference baseline | `my_model/model_example.py` | Power-law fade + Arrhenius fit; **defines score = 1.0** |
| Model switch | `my_model/__init__.py` | One line selects example vs our model |
| Dry-run data | `sample_data/` (2 synthetic cells), `sample_input.csv` | Structural smoke test only |
| Real dataset | `data/` (6 cells, ~560 MB, tracked via Git LFS) | The fine-tuning set |
| Challenge PDF | `data/TechArena_2026_Topic_1_Challenge1.pdf` | Full task description + open-data links |

## 3. The interface we must implement (the ONLY thing that matters for scoring)

```python
class MyModel:
    def fit(self, cells):
        # cells: list of Cell objects -> cell.temperature_degC, cell.c_rate,
        #        cell.soh (DataFrame [cycle_number, soh_percent]),
        #        cell.time_series() -> raw signals (lazy; heavy)
        # learn everything onto self.*  -> whole object is pickled afterwards

    def predict_soh(self, temperature_degC, c_rate):
        # return 1D array: SOH % for cycles 1, 2, 3, ... (12000 values is always safe)
```

The evaluation calls ONLY these two methods. `fit()` runs once; `predict_soh(T, C)`
is called per operating point in a possibly separate process. At prediction time
there is NO cell data — only (temperature, C-rate). Anything learned from raw
signals must be baked into model parameters during `fit()`.

## 4. Verified data facts (measured, not assumed)

Raw time-series columns (30 s sampling): `cycle_number, step_type,
time_in_cycle_s, voltage_V, current_A, temperature_C, step_capacity_Ah,
absolute_time`. Protocol: 30 min rest → CC-CV charge to 3.649 V → 30 min rest →
CC discharge to 2.5 V. `time_in_cycle_s` can be negative (rest rows before cycle
start). SOH label = max cumulative `step_capacity_Ah` of the `cc_discharge`
step / 102 Ah × 100; implausible (>115%) and aborted partial cycles → NaN.

Per-cell SOH statistics (from the official `framework.data` derivation):

| cell_id                  | T (°C) | C-rate | cycles | SOH start | SOH end | min | reached 70%? |
|--------------------------|-------:|-------:|-------:|----------:|--------:|----:|--------------|
| 102Ah_25degC_0p5C_cell3  | 25     | 0.5    | 1375   | 102.1     | 93.1    | 93.1 | NO (stopped early) |
| 102Ah_25degC_1C_cell3    | 25     | 1.0    | 5307   | 102.5     | 66.3    | 66.3 | YES |
| 102Ah_35degC_1C_cell1    | 35     | 1.0    | 5356   | 102.1     | 72.6    | 54.5 | YES (deep knee) |
| 102Ah_45degC_0p5C_cell3  | 45     | 0.5    | 4501   | 103.0     | 58.7    | 42.8 | YES (deepest) |
| 102Ah_45degC_1C_cell1    | 45     | 1.0    | 2396   | 102.6     | 84.8    | 84.8 | NO (stopped early) |
| 102Ah_55degC_1C_cell3    | 55     | 1.0    | 2415   | 102.8     | 65.7    | 65.7 | YES |

Key takeaways:
- **Only 6 training conditions**: (25,0.5) (25,1) (35,1) (45,0.5) (45,1) (55,1).
  The corners **35 °C/0.5C and 55 °C/0.5C are missing** — almost certainly in the
  hidden test set, plus interpolated points anywhere in 25–55 °C × 0.5–1.0 C.
- Curves are highly nonlinear (knees at 45 °C/0.5C and 35 °C/1C); a straight
  line or single power law will lose badly near the knee.
- Two cells stop above 70% (93.1, 84.8) — organizers say "last available SOH
  counts" for those conditions; the evaluation input CSV decides the cycles.
- **Transfer is the core of the challenge**: the provided 6 cells alone cannot
  pin down a 2-D temperature×rate response surface (only 2 of 4 temperatures
  have both C-rates). Open LFP pre-training is explicitly expected.

## 5. How the framework behaves (verified from code)

- `python run_model.py --model train --input <dataset-folder> --output-dir <dir>`
  → loads all cells (recursive glob for `*_time_series*part01.csv`, cell folder
  name must match `102Ah_<T>degC_<0p5C|1C>_cell<N>`), calls `fit(cells)`, pickles
  the whole model object to `model_state/model.pkl`.
- `--model test/valid --input <input.csv>` → input CSV columns `Cycle`,
  `Temperature (degC)`, `Current (C-rate)` (optional `SOH_true`); framework
  calls `predict_soh(T, C)` once per (T,C) group, indexes `soh[cycle-1]`, and
  writes `output.csv` (columns: Cycle, SOH_true, Temperature (degC),
  Current (C-rate), SOH_model, errSOH).
- Hard validation in `framework/io.py`: `predict_soh` must be long enough for
  the max requested cycle (12000 is always safe), all values finite, and in
  **(0, 120]** — else ValueError.
- Pickle covers numpy/scipy/sklearn/PyTorch. If the model can't be pickled,
  implement `save(folder)` / `load(folder)` (see `framework/persistence.py`).
- `validate_submission.py` = same intake checks as official pipeline:
  structure, requirements readable, train+test dry run on sample data, output
  schema. Prints `SUBMISSION READY`. Optional `--data-dir <folder>` does a full
  run on the real dataset (already verified: passes, ~3 s with the example).

## 6. How we get scored (what to optimize)

1. Hidden ground truth: sibling cells of the 6 released conditions + cells at
   unseen operating points within 25–55 °C × 0.5–1.0 C, cycled deep through
   the knee.
2. A **fixed composite error metric, not published**, reported **relative to the
   example baseline: baseline = 1.0, lower is better**. Leaderboard shows this.
3. Ranking = leaderboard score + our report (method, validation methodology,
   datasets used with citations/licenses) + reproducibility.
4. **Data-efficiency bonus**: every submission is re-trained automatically with
   fewer cells and only early-life cycles. Models that keep accuracy with less
   data earn a bonus. → Design `fit()` to degrade gracefully (regularization,
   priors from pre-training, not memorizing full curves).

## 7. Hard constraints & pitfalls

- ONLY `my_model/` is ours. `framework/`, `run_model.py`, `validate_submission.py`
  must stay byte-identical (organizers use their own copies anyway).
- Flip `my_model/__init__.py` to `ActiveModel = MyModel` or we silently submit
  the baseline example.
- No `raise NotImplementedError` left in the active model.
- No absolute/team-specific paths; relative paths only for files shipped in
  `my_model/`. No internet during scoring — ship pretrained weights inside
  `my_model/`.
- `predict_soh` must be finite, in (0, 120], ≥ 12000 values, and must actually
  depend on (T, C) — a condition-independent model scores ~1.0 at best.
- Random seeds fixed; every dependency pinned in `requirements.txt`.
- Runtime limits: 2 h train (full set) on 96-core/554 GB RAM Windows Server
  with an RTX PRO 94 GB GPU; 30 min per cell for test. Local sanity limits in
  validate_submission.py (900 s/300 s) are NOT the competition limits.
- Real-data quirks to expect: missing temperature sensor rows (empty
  `temperature_C`), `logic_test` verification segments, boundary partial cycles
  (flagged, not dropped), parts must be concatenated in order.

## 8. The actual scientific problem (why it's hard)

- **Extrapolation in 2-D operating space** from 6 points → the response surface
  (Arrhenius-like in T, power-law-ish in C-rate, plus knee dynamics) is
  underdetermined. The baseline's linear-in-1/T, linear-in-ln(C) fit is a
  starting point, not the answer.
- **Knee prediction**: the fastest-fading cells (45 °C/0.5C → 42.8 %, 35 °C/1C →
  54.5 %) roll over sharply. Missing the knee costs far more than a small
  slope error.
- **Transfer domain gap**: public LFP data (A123 18650 1.1 Ah, small cylindrical,
  fast-charge) vs our 102 Ah prismatic symmetric CC-CV. Formats, thermal
  behavior and possibly knee mechanisms differ — naive transfer adds noise.
- Two cells never reach 70 % → their conditions can only be scored where data
  exists; prediction quality there still matters (they're part of the held-out
  siblings).

## 9. Public pre-training data (from the PDF — must cite + be reproducible)

1. Wheeler et al., "Aging study on twenty A123 18650 Graphite/LFP 1.1 Ah cells"
   (2025), UMR AMPERE.
2. Che et al., Cell Reports Physical Science 4.12 (2023) — continual learning
   battery aging: data.mendeley.com/datasets/n3b54nsw8m/9.
3. Catenaro & Onori, Data in Brief 35 (2021) 106894 — galvanostatic discharge
   at different rates/temperatures (Mendeley).

Any additional open dataset is allowed if (a) cited with license in the report
and (b) reproducible via download script or declared cached copy. Performance
on open data is NOT scored — it is only a means to a better transfer.

## 10. Validation strategy (part of the assessed work)

Official metric is unpublished → we must design our own and defend it:
- **Leave-one-condition-out (LOCO)**: train on 5 of 6 conditions, predict the
  6th; report error per condition and overall. This directly mimics the
  held-out siblings.
- **Corner extrapolation check**: hold out (25,0.5) or (55,1) and see how
  badly extreme corners extrapolate — closest proxy to the unseen
  (35,0.5)/(55,0.5) points.
- **Interpolation check**: synthesize/interpolate mid points (30 °C, 0.7 C,
  40 °C, 0.75 C) and check smoothness; the baseline itself treats them as
  continuous.
- **Knee/end-of-life error**: weight errors below ~85 % SOH heavily, since
  reaching-70 % timing is the headline deliverable.
- Keep the validation script in-repo (e.g. `eval/`) so the report can cite it.

## 11. Plan of attack (phases)

1. **EDA** (next): per-cycle feature extraction from `time_series()` — capacity
   curves, ICA/dQ-dV peaks, DC resistance at rest→discharge transition, CV-phase
   time, temperature midpoints; correlate early features with knee timing.
2. **Baseline beat**: implement a better parametric family (e.g. bi-exponential
   or sigmoid-knee fade `SOH(n) = (100-c)·exp(-(n/τ)^β)+c`, fit per cell, then
   regress parameters on T and C-rate; uncertainty-aware). Beat the example on
   LOCO before touching ML.
3. **Signal features**: build the per-cell feature table; check if early-life
   features (first 50–200 cycles) predict long-term fade — this is also what
   the reduced-budget bonus rewards.
4. **Open-data pre-training**: download the 3 linked datasets (offline scripts),
   pre-train a curve/feature encoder; ship weights in `my_model/`.
5. **Transfer/fine-tune**: fine-tune on the 6 cells with strong regularization
   (priors anchored to pre-trained response surface; physics-informed
   constraints e.g. Arrhenius monotonicity in T).
6. **Validation harness + report**: LOCO numbers, held-out corners, knee error;
   write the README approach section with citations.
7. **Submission**: fresh venv from requirements.txt → `validate_submission.py`
   → zip as `<TeamName>_Challenge1.zip` → submit.

## 12. Operational notes (this repo)

- Dataset is in `data/`, tracked via Git LFS (collaborators: clone + `git lfs pull`).
- venv: `.venv` created with `uv` (system python3.14 lacks ensurepip).
  Recreate with: `uv venv .venv && uv pip install --python .venv/bin/python -r requirements.txt`.
- Current state: example model active (`__init__.py`), all validation passes.
- Don't commit `model_state/`, `.val_out*/`, `__pycache__/` (already ignored).
