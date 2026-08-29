# research.md — SOTA journal papers for Challenge 1 (SOH fade prediction)

Scope: journal-published (and a few high-impact preprint) works on battery
SOH/lifetime prediction, early prediction, knee-point identification, and
transfer learning — ranked by how directly they map onto OUR task:
predict SOH-vs-cycle curves to 70 % (knee included) for unseen operating
points in 25–55 °C × 0.5–1.0 C, fine-tuned from only 6 cells, pre-trained
on open LFP data.

Compiled 2026-08-29 from web research. DOIs/links included where available.

---

## Tier 1 — Directly on-task: lifetime prediction under diverse / unseen conditions

### 1. BatLiNet — inter-cell deep learning  ⭐ most relevant
Zhang, H. et al. "Battery lifetime prediction across diverse ageing conditions
with inter-cell deep learning." *Nature Machine Intelligence* 7, 270–277 (2025).
https://www.nature.com/articles/s42256-024-00972-x  (arXiv:2310.05052)

- Problem: predict cycle life across varied ageing conditions (protocols,
  temperatures, even chemistries) — the closest published analog to our task.
- Method: **inter-cell learning** — an auxiliary task that predicts the
  *lifetime difference* between a target cell and a reference cell, using
  cycle-level V–Q (discharge) features, combined with conventional single-cell
  learning. Intra-cell differences (Severson-style Δfeatures between early
  cycles) + inter-cell differences against a shared reference cell.
- Results: cuts MAPE by >40 % vs the single-cell CNN counterpart, averaged
  over MATR-1, MATR-2, HUST (LFP) and MIX (cross-chemistry) benchmarks.
  Transfers across chemistries (trained on LFP → predicts NMC/LCO/NCA).
- Why it matters for us: the "reference cell" trick is a cheap way to
  regularize with only 6 target cells; HUST/MATR are LFP like our 102 Ah cell.
- Caveat: predicts a scalar lifetime (cycles to 80 %/90 %), not the full
  SOH-vs-cycle curve — we need the curve, so adapt the *contrastive idea*.

### 2. Degradation-informed features + hierarchical Bayesian extrapolation
Li, T., Zhou, Z., Thelen, A., Howey, D. A., Hu, C. "Predicting battery
lifetime under varying usage conditions from early aging data."
*Cell Reports Physical Science* 5, 101891 (2024).
https://doi.org/10.1016/j.xcrp.2024.101891 (arXiv:2307.08382)

- Problem: lifetime prediction when cells are cycled under *varying*
  charge/discharge rates (0.5–3 C) and DoDs (4–100 %) — i.e. extrapolation
  across usage conditions, like our unseen operating points.
- Data: NEW public dataset, 225 NMC/graphite cells
  (https://doi.org/10.25380/iastate.22582234).
- Method: early-life features from Q(V) and incremental-capacity dQ/dV curves
  that quantify degradation modes (loss of active material), fed to elastic
  net and a **hierarchical Bayesian model (HBM)** that shares strength across
  cells/groups.
- Results: 15.1 % MAPE in-distribution (elastic net); **21.8 % MAPE
  out-of-distribution (HBM)** — the OOD number is the one that matters for us.
- Why it matters: shows (a) dQ/dV-derived degradation features carry
  lifetime signal beyond Severson's var(ΔQ) feature, (b) a hierarchical
  Bayesian structure is the demonstrated way to extrapolate to unseen
  conditions with few cells. Both ideas transfer directly.

### 3. Discovery Learning — minimal experiments, industrial cells
Zhang & Zhang et al. "Discovery Learning predicts battery cycle life from
minimal experiments." *Nature* 650, (2026).
https://www.nature.com/articles/s41586-025-09951-7

- Predicts cycle life from only ~50 cycles (a few days of data) on
  123 industrial large-format pouch cells across diverse material/design
  combinations and cycling protocols.
- Method: deliberately designed "discovery" experiments + data-driven
  modelling — the data-generation strategy is part of the model.
- Why it matters: strong recent evidence that early-cycle (≤100 cycles)
  information suffices for long-horizon life prediction, and that
  cross-design/chemistry transfer is achievable with the right inductive
  biases. Conceptually validates our "predict full curve from limited
  fine-tuning data + pretrained priors" plan.

---

## Tier 2 — Foundational: early prediction & the benchmark dataset

### 4. Severson et al. — THE reference benchmark (LFP!)
Severson, K. A. et al. "Data-driven prediction of battery cycle life before
capacity degradation." *Nature Energy* 4, 383–391 (2019).
https://www.nature.com/articles/s41560-019-0356-8

- 124 commercial A123 **Graphite/LFP 1.1 Ah 18650** cells (the same cell type
  as open dataset [1] in our challenge PDF), 72 fast-charging policies,
  cycled to 80 % EOL.
- Method: discharge-curve "sagging" features — variance of ΔQ_{100−10}(V) —
  plus other early-cycle features → elastic-net regression predicts cycle
  life with ~9 % test error using only the first 100 cycles.
- Why it matters: this dataset+feature recipe IS the pre-training corpus our
  challenge expects (PDF dataset [1] is the A123 20-cell extension). The
  ΔQ(V) feature family is the proven baseline transfer feature set.
- Follow-up: Attia, P. M. et al. *Nature* 578, 397–402 (2020) used the same
  features for closed-loop fast-charging optimization — shows the features
  are robust across protocols.

### 5. Statistical learning re-analysis
Severson et al. / follow-up (Attia group). "Statistical learning for accurate
and interpretable battery lifetime prediction." arXiv:2101.01885 (2021).
- Re-examines the 2019 benchmark: which features/models actually generalize,
  and pitfalls of small test sets. Good methodological checklist for our
  LOCO validation design (feature selection stability, split leakage).

---

## Tier 3 — Knee-point & nonlinear aging (our "knee included" requirement)

### 6. Knee-point identification + ML prediction
Fermín-Cueto, P., McTurk, E., Allerhand, M., Medina-Lopez, E., Anjos, M. F.,
Sylvester, J., dos Reis, G. "Identification and machine learning prediction
of knee-point and knee-onset in capacity degradation curves of lithium-ion
cells." *Energy and AI* 1, 100006 (2020).
https://doi.org/10.1016/j.egyai.2020.100006

- Robust, application-independent knee-point and knee-onset identification
  method; ML prediction of knee position from early data.
- Why it matters: gives us the off-the-shelf tool to (a) label knees in the
  6 training cells, (b) define an auxiliary prediction target (knee cycle)
  to improve curve forecasts around the knee, (c) evaluate knee error
  separately in our validation.

### 7. Curvature-based knee-onset identification
"Battery Capacity Knee-Onset Identification and Early Prediction Using
Degradation Curvature." arXiv:2304.11671 (v2, 2024).
- Knee-onset via degradation curvature; outperforms the double Bacon–Watts
  model on three datasets (which fails outright on one). Knee-onset
  correlates with EOL and gives earlier warning.
- Preprint — likely journal-published by now; check for the final venue.

### 8. Knee interval / nonlinear aging prediction
"Early prediction of battery knee interval before capacity degradation."
*Journal of Power Sources* (2025). https://doi.org/10.1016/j.jpowsour.2025.xxxx
(see ScienceDirect S0378775325002745)
- Frames the sharp-fade regime as a "knee interval" and predicts it early;
  ties knee onset to SEI-driven mechanisms.

---

## Tier 4 — Transfer learning for SOH (small samples, cross-domain)

### 9. Transfer-learning meta-analysis (survey)
"Transfer learning for battery health estimation: a comprehensive
meta-analysis of models, strategies, and domain transfer scenarios."
*Ionics* (Springer, 2026). https://link.springer.com/article/10.1007/s11581-026-07030-z
- Systematic comparison of TL strategies (fine-tune, domain adaptation,
  self-supervised pretraining) across source→target scenarios. Use it to
  pick the right TL recipe for open-LFP → 102 Ah transfer.

### 10. Deep TL for SOH under fast charging
"Battery state of health estimation under fast charging via deep transfer
learning." (2025, open access, PMC12033934).
- Hybrid DNN; transfer learning cut RMSE 0.00741 → 0.00109 and lifted
  R² 0.986 → 0.998 on a different-chemistry target. Concrete evidence that
  fine-tuned TL beats from-scratch on small target sets.

### 11. Small-sample TL across chemistries/protocols/temperatures
"Deep transfer learning enabled online state-of-health estimation of
lithium-ion batteries under small samples across different cathode
materials, ambient temperature and charge-discharge protocols."
*Journal of Power Sources* (2025). S0378775325013394
- Pretrain on source chemistry, fine-tune with a small labeled slice of the
  target task. Exactly our fine-tune step.

### 12. Self-supervised pretraining (SSF-WL)
Wang et al. (2024), cited in arXiv:2402.00068 — self-supervised pretraining
on unlabeled source battery data, then small-sample fine-tune on target.
- If the open LFP corpora are large but heterogeneous (different protocols),
  self-supervised curve/feature pretraining is a strong option before
  fine-tuning on the 6 cells.

### 13. Heterogeneous-feature few-shot TL
"Bridging the feature gap: Heterogeneous transfer learning for lithium-ion
battery health estimation using one-shot data." (2026). S2590123026002069
- MMD-based heterogeneous-feature transfer for one-shot/few-shot SOH.
  Relevant if our feature spaces (18650 fast-charge vs prismatic CC-CV)
  don't align; MMD alignment may still let us share pretrained features.

---

## Tier 5 — Physics-informed & semi-empirical (response surface in T × C-rate)

### 14. Arrhenius behavior of aging vs T, C-rate, and age
Kucinskis, G. et al. "Arrhenius plots for Li-ion battery ageing as a function
of temperature, C-rate, and ageing state – An experimental study."
*Journal of Power Sources* 549, 232129 (2022).
https://doi.org/10.1016/j.jpowsour.2022.231xxx (S0378775322011065)

- Key finding: the dominant aging mechanism (and thus the "optimum"
  temperature) *changes* with C-rate, SOH, and cell design — a single
  Arrhenius term is not enough. This is a direct warning for our
  T×C-rate extrapolation: the example baseline's single Arrhenius slope
  will fail at corners like 55 °C/0.5 C.
- Why it matters: motivates mechanism-aware parametric forms (e.g. two
  competing Arrhenius terms, C-rate-dependent activation energy) instead of
  one linear-in-1/T fit.

### 15. Physics-informed curve reconstruction (Arrhenius + self-attention)
"Optimizing Cycle Life Prediction of Lithium-ion Batteries via a
Physics-Informed Model." arXiv:2404.17174 (2024).
- Fits capacity-loss curves to an Arrhenius-inspired equation
  Q_loss(n) = C + A·exp(−B·n^β) (per-cell), then a self-attention model
  predicts the curve parameters from early-cycle features. Predicts the
  FULL capacity-loss curve (not just EOL), so any EOL threshold works.
- Why it matters: this is almost exactly our required output shape
  (full SOH-vs-cycle curve, any threshold down to 70 %). Strong template
  for our Phase-2 parametric model.

### 16. Semi-empirical vs symbolic regression (calendar aging)
Onori group, *J. Electrochem. Soc.* (2025), comparing semi-empirical,
symbolic-regression and other models on a long-term calendar-aging dataset
(Lam et al., *Joule* 9, 146 (2025)).
- Useful benchmark of how far simple interpretable parametric models go
  vs data-driven ones when extrapolating in stress factors.

---

## Tier 6 — Foundation models / generative (frontier, 2025–2026)

### 17. Pretrained battery transformer (PBT)
arXiv:2512.16334 (Dec 2025). Foundation model for battery life prediction;
battery-knowledge-encoded mixture-of-experts to learn from scarce,
heterogeneous lifetime data. (Preprint.)

### 18. BatteryGPT
Hu, Fu et al. "Early prediction of lithium-ion battery degradation with a
generative pre-trained transformer." *Nature Communications* (2025).
https://www.nature.com/articles/s41467-025-66819-0
- Two-stage GPT: autoregressively predicts future charging data and SOH
  from early cycles. Generative roll-out of the degradation trajectory.

### 19. DegradAI
Sudarshan, Vajja et al. "DegradAI: A scalable framework for early battery
health prediction." Nature portfolio journal (2025).
https://www.nature.com/articles/s44406-025-00008-2
- Chemistry-aware deep learning predicting long-term capacity fade from
  ~5 h of early-cycle data across chemistries and operating conditions.

---

## Open datasets worth using for pre-training (LFP-first)

| Dataset | Chemistry | Cells | Why |
|---|---|---|---|
| Severson/MATR (Nature Energy 2019) | LFP 1.1 Ah 18650 | 124 | THE benchmark; same cell as PDF dataset [1] |
| Wheeler et al., *Scientific Data* 2025 (s41597-025-04712-7, HAL hal-04980712) | LFP 1.1 Ah 18650 | 20, aged to 40 % loss (first+second life) | PDF dataset [1]; long-fade coverage incl. knee |
| HUST battery aging dataset | LFP | ~30 | LFP, different protocol — good domain-shift test |
| Li et al. 2024 (Iowa State, doi:10.25380/iastate.22582234) | NMC | 225 | Wide condition coverage for pretraining response surface |
| Che et al. (Mendeley n3b54nsw8m/9) | ? | large | PDF dataset [2] |
| Catenaro & Onori (Data in Brief 2021) | 3 cells | 3 | PDF dataset [3] (galvanostatic, rates/temps) |

---

## What this means for OUR plan (synthesis)

1. **Feature side**: Severson-style ΔQ(V) features + Li et al.'s dQ/dV
   degradation-mode features are the proven early-life predictors. Our raw
   time series (voltage, current, temperature, capacity per 30 s) is rich
   enough for both; ICA/dQ-dV is directly computable.
2. **Model side**: predict the whole curve, not a scalar — follow the
   physics-informed curve-parameter approach (#15) or a hierarchical
   Bayesian curve model (#2). BatLiNet's inter-cell contrast (#1) is the
   best trick for squeezing signal out of 6 cells.
3. **Extrapolation side**: expect Arrhenius nonlinearity (#14) — use
   mechanism-aware forms and validate LOCO, paying special attention to the
   missing corners (35 °C/0.5 C, 55 °C/0.5 C).
4. **Transfer side**: pretrain on Severson/Wheeler/HUST LFP (same chemistry
   as target), fine-tune on the 6 cells; self-supervised pretraining (#12)
   if feature alignment is an issue; MMD alignment (#13) as a fix.
5. **Knee side**: use Fermín-Cueto (#6) knee labels as an auxiliary target
   and a separate evaluation axis; the 45 °C/0.5 C cell (knee at ~43 % SOH)
   is our hardest case.
6. **Benchmark honestly**: 9 % test error (Severson) / 21.8 % OOD MAPE
   (Li et al.) / >40 % MAPE reduction (BatLiNet) are the numbers to beat in
   our own LOCO validation — but the challenge's relative-to-baseline
   metric is what ultimately counts.
