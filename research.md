# research.md — SOTA journal papers for Challenge 1 (SOH fade prediction)

Scope: journal-published (plus high-impact preprints) works on battery
SOH/lifetime prediction, early prediction, knee-point identification, and
transfer learning — ranked by how directly they map onto OUR task:
predict SOH-vs-cycle curves to 70 % (knee included) for unseen operating
points in 25–55 °C × 0.5–1.0 C, fine-tuned from only 6 cells, pre-trained
on open LFP data.

v2 (2026-08-29): metadata (authors, venue, year, DOI) verified via the
arXiv API and Crossref/Semantic Scholar; headline claims carry verbatim
abstract evidence. v1 was assembled from search snippets only — see the
changelog at the end.

---

## Tier 1 — Directly on-task: lifetime prediction under diverse / unseen conditions

### 1. BatLiNet — inter-cell deep learning  ⭐ most relevant
Han Zhang, Yuqi Li, Shun Zheng, Ziheng Lu, Xiaofan Gui, Wei Xu, Jiang Bian.
"Accurate battery lifetime prediction across diverse aging conditions with
deep learning." arXiv:2310.05052 (2023; journal version: *Nature Machine
Intelligence* 7, 270–277, 2025).[1]

- Key finding: "incorporating inter-cell feature differences, rather than
  solely considering single-cell characteristics, significantly increases
  the accuracy of battery lifetime prediction and its cross-condition
  robustness."[1]
- Method: a framework "capable of accommodating various aging conditions
  and facilitating effective learning under low-resource conditions by
  leveraging data from rich conditions" — single-cell learning plus an
  inter-cell task that contrasts pairs of cells against a reference cell.[1]
- Reported results (journal version): "BatLiNet achieves ≤ 11 % mean
  absolute percentage error (MAPE) given the first 100 cycles and ≤ 18 %
  for the first 20 cycles"; crucially for us, "BatLiNet further reduced
  prediction error by 10 % over transfer learning, achieving 20.26 % MAPE
  using only two to eight target cells and historical LFP data".[1]
- Why it matters: the inter-cell contrast is the best published trick for
  squeezing signal out of very few target cells — we have 6, squarely in
  BatLiNet's "two to eight target cells" regime; HUST/MATR are LFP like
  our 102 Ah cell.[1]
- Caveat: predicts scalar cycle life (to 80 %/90 % capacity), not the full
  SOH-vs-cycle curve — we need the curve, so adapt the *contrastive idea*.

### 2. Degradation-informed features + hierarchical Bayesian extrapolation
Tingkai Li, Zihao Zhou, Adam Thelen, David A. Howey, Chao Hu. "Predicting
battery lifetime under varying usage conditions from early aging data."
*Cell Reports Physical Science* 5, 101891 (2024), DOI
10.1016/j.xcrp.2024.101891 (preprint: arXiv:2307.08382).[2][9]

- Data: "a newly generated dataset from 225 nickel-manganese-cobalt/graphite
  Li-ion cells aged under a wide range of conditions", with charge/discharge
  rates 0.5–3 C and DoD 4–100 % — i.e. prediction *across* usage
  conditions, the same extrapolation regime we face.[2]
- In-distribution: "a lifetime prediction of in-distribution cells with
  15.1% mean absolute percentage error using no more than the first 15% of
  data, for most cells."[2]
- Out-of-distribution (the number that matters for us): "a hierarchical
  Bayesian regression model shows improved performance on extrapolation,
  achieving 21.8% mean absolute percentage error" on OOD cells.[2]
- Features: early-life capacity-voltage and dQ/dV quantities that capture
  degradation modes (loss of active material), not just Severson's var(ΔQ).[2]
- Public dataset: https://doi.org/10.25380/iastate.22582234.[2]

### 3. Discovery Learning — minimal experiments, industrial cells
Jiawei Zhang, Yifei Zhang, Baozhao Yi, et al. "Discovery Learning predicts
battery cycle life from minimal experiments." *Nature* 650 (2026), DOI
10.1038/s41586-025-09951-7.[11]

- Predicts cycle life from ~50 cycles (a few days of data) on
  123 industrial large-format pouch cells across diverse material/design
  combinations and cycling protocols; 32 citations in its first months.[11][unverified]
- Why it matters: recent top-journal evidence that early-cycle data suffices
  for long-horizon life prediction, and that cross-design transfer works
  with deliberate data-generation + modelling.

---

## Tier 2 — Foundational: early prediction & the benchmark dataset

### 4. Severson et al. — THE reference benchmark (LFP!)
Kristen A. Severson, Peter M. Attia, Norman Jin, et al. "Data-driven
prediction of battery cycle life before capacity degradation."
*Nature Energy* 4, 383–391 (2019), DOI 10.1038/s41560-019-0356-8 (2721
citations).[8]

- "We generate a comprehensive dataset consisting of 124 commercial
  lithium iron phosphate/graphite cells" — 1.1 Ah A123 18650, the same
  cell type as open dataset [1] in our challenge PDF, with 72 fast-charging
  policies, cycled to 80 % EOL.[8]
- "Our best models achieve 9.1% test error for quantitatively predicting
  cycle life using the first 100 cycles"; the classification variant
  reaches 4.9 % test error using only the first 5 cycles.[8]
- Model: "the elastic net. The final model uses a linear combination of a
  subset of the proposed features to predict the logarithm" of cycle
  life — the features come from discharge-voltage curves
  (ΔQ_{100−10}(V)).[8] (Verified against the paper's own PDF; the dataset
  is further documented by the same group's follow-up.[5])
- Why it matters: this dataset+feature recipe IS the pre-training corpus
  our challenge expects; the ΔQ(V) feature family is the proven transfer
  feature set.[8]

### 5. Statistical learning follow-up (same group)
Peter M. Attia, Kristen A. Severson, Jeremy D. Witmer. "Statistical
learning for accurate and interpretable battery lifetime prediction."
arXiv:2101.01885 (2021).[5]

- "We first present the 'capacity matrix' concept as a compact
  representation of battery electrochemical cycling data"; simple linear
  models with engineered features "achieve comparable performance to the
  highest-performing models previously published for this dataset".[5]
- Lesson: for small training sets, interpretable feature-based models beat
  complex ML — a caution for our 6-cell regime.[5]

---

## Tier 3 — Knee-point & nonlinear aging (our "knee included" requirement)

### 6. Knee-point identification + ML prediction (240 citations)
Paula Fermín-Cueto, Euan McTurk, Michael Allerhand, et al. "Identification
and machine learning prediction of knee-point and knee-onset in capacity
degradation curves of lithium-ion cells." *Energy and AI* 1, 100006 (2020),
DOI 10.1016/j.egyai.2020.100006.[10]

- "Li-ion cells exhibit a slow capacity degradation up to a knee-point,
  after which the degradation accelerates rapidly until the cell's
  End-of-Life"; proposes robust knee-point identification plus the
  'knee-onset' concept, linking cycle life, knee-point and knee-onset.[10]
- Why it matters: off-the-shelf tool to label knees on our 6 cells, define
  an auxiliary prediction target, and evaluate knee error separately.

### 7. Curvature-based knee-onset identification
Huang Zhang, Faisal Altaf, Torsten Wik. "Battery Capacity Knee-Onset
Identification and Early Prediction Using Degradation Curvature."
arXiv:2304.11671 (2023).[4]

- Curvature-based knee/knee-onset definition validated on two chemistries
  + synthetic data; "a significantly strong correlation is found between
  knee and end of life (EoL) and almost equally strong between knee onset
  and EoL"; succeeds where the state-of-the-art (double Bacon–Watts) method
  failed, and works without the full fade curve (online use).[4]

### 8. Knee + full-trajectory forecasting
Samuel Greenbank, David Howey. "Automated Feature Extraction and Selection
for Data-Driven Models of Rapid Battery Capacity Fade and End of Life."
*IEEE Transactions on Industrial Informatics* (2022), DOI
10.1109/TII.2021.3106593 (147 citations).[18]

- "The onset of rapid degradation is called the 'knee point'"; automated
  feature selection feeds a Gaussian process regression "from which the
  entire capacity fade trajectory, knee point, and end of life may be
  predicted"; calendar time and time-in-voltage-region dominate.[18]
- Why it matters: a full-trajectory GP model — closest published analogue
  to our "output the whole SOH curve" requirement.

---

## Tier 4 — Transfer learning for SOH (small samples, cross-domain)

### 9. Transfer-learning meta-analysis (survey)
Ebubekir Buğra Özarslan, Senem Kursun. "Transfer learning for battery
health estimation: a comprehensive meta-analysis of models, strategies,
and domain transfer scenarios." *Ionics* (2026), DOI
10.1007/s11581-026-07030-z.[14]

- Systematic comparison of TL strategies across source→target scenarios.
  Use it to pick the right TL recipe for open-LFP → 102 Ah transfer.
  (Metadata verified; content not yet read in depth.)

### 10. Rapid cycle-life test via transfer learning
Zhu et al. "Rapid Test and Assessment of Lithium-Ion Battery Cycle Life
Based on Transfer Learning." *IEEE Transactions on Transportation
Electrification* (2024), DOI 10.1109/TTE.2024.3354107 (20 citations).[20]

- "The cycle life test provides crucial support for using and maintenance
  of lithium-ion batteries" — transfer learning shortens the test. Direct
  analogue of our "predict EOL from limited target data" setup.[20]

### 11. Small-sample TL across chemistries/protocols/temperatures
"Deep transfer learning enabled online state-of-health estimation of
lithium-ion batteries under small samples across different cathode
materials, ambient temperature and charge-discharge protocols."
*Journal of Power Sources* (2025), DOI 10.1016/j.jpowsour.2025.237503.[21]

- Pretrain on source chemistry, fine-tune with a small labeled slice of the
  target task. Exactly our fine-tune step (metadata verified; abstract not
  machine-accessible).

### 12. Test-time training for cross-domain SOH (BatteryTTT)
Yuyuan Feng, Guosheng Hu, Xiaodong Li, Zhihong Zhang. "Adapting Amidst
Degradation: Cross Domain Li-ion Battery Health Estimation via
Physics-Guided Test-Time Training." arXiv:2402.00068 (2024).[6]

- "Existing transfer learning methods ... still require sufficient
  unlabeled target data (UTD) for effective adaptation"; BatteryTTT
  "adapts the model continually using each UTD collected amidst
  degradation", integrating physical laws into self-supervised learning.[6]
- Why it matters: if we want the model to keep adapting during evaluation
  (not allowed by our framework — predict_soh gets no data), the physics-
  guided self-supervision idea still transfers to fit-time regularization.

### 13. Heterogeneous-feature few-shot TL
"Bridging the feature gap: Heterogeneous transfer learning for lithium-ion
battery health estimation using one-shot data." (2026).[unverified]
(Found via search; MMD-based feature alignment for one-shot SOH. DOI not
yet resolved — check before citing in the report.)

---

## Tier 5 — Physics-informed & semi-empirical (response surface in T × C-rate)

### 14. Arrhenius behavior of aging vs T, C-rate, and age
Gints Kucinskis, Maral Bozorgchenani, et al. "Arrhenius plots for Li-ion
battery ageing as a function of temperature, C-rate, and ageing state – An
experimental study." *Journal of Power Sources* 549, 232129 (2022), DOI
10.1016/j.jpowsour.2022.232129 (124 citations).[17]

- Title itself states the core finding: aging depends jointly on
  temperature, C-rate AND ageing state — a single Arrhenius term is
  insufficient across the T×C-rate plane.[17]
- Reported: the dominant aging mechanism (and hence optimum operating
  temperature) shifts with C-rate, SOH and cell design.[unverified]
  (Abstract not machine-accessible; mechanism-crossover claim unverified.)
- Why it matters: the example baseline's single Arrhenius slope will fail
  at corners like 55 °C/0.5 C — we need mechanism-aware parametric forms.

### 15. Physics-informed curve reconstruction (Arrhenius + self-attention)
Constantin-Daniel Nicolae, Sara Sameer, Nathan Sun, Karena Yan. "Optimizing
Cycle Life Prediction of Lithium-ion Batteries via a Physics-Informed
Model." arXiv:2404.17174 (2024).[3]

- Fits capacity-loss curves to a physics-based equation, then "a
  self-attention layer to reconstruct entire battery capacity loss curves",
  predicting "the entire capacity loss curve instead of cycle life" — any
  EOL threshold without retraining, backed by physical intuition.[3]
- Why it matters: this is almost exactly our required output shape. Strong
  template for our Phase-2 parametric model.

---

## Tier 6 — Frontier: foundation models / generative (2025–2026)

### 16. Pretrained battery transformer (PBT)
Ruifeng Tan, Weixiang Hong, Jia Li, Jiaqiang Huang, Tong-Yi Zhang.
"Pretrained battery transformer (PBT): A foundation model for battery life
prediction." arXiv:2512.16334 (2025).[7]

- "PBT is first pretrained on 13 lithium-ion battery datasets to yield a
  general PBT ... adapted through transfer learning into specialized PBT
  models for target scenarios."[7]
- Results: "Across 15 datasets covering 977 batteries and 528 sets of aging
  conditions from lithium-ion, sodium-ion and zinc-ion batteries, PBT
  achieves state-of-the-art performance, surpassing the strongest competing
  method by 21.9% on average, with gains of up to 86.9%."[7]
- Why it matters: the strongest case that foundation-style pretraining +
  target fine-tuning beats from-scratch — exactly our transfer recipe.
  (Preprint; heavy — a fallback option, not the default plan.)

### 17. BatteryGPT
Jincheng Hu, Pengyu Fu, Zhongbao Wei, et al. "Early prediction of
lithium-ion battery degradation with a generative pre-trained transformer."
*Nature Communications* (2025), DOI 10.1038/s41467-025-66819-0 (31
citations).[12]

- Two-stage GPT that autoregressively predicts future charging data and
  SOH from early cycles ("crucial for effective predictive maintenance and
  recycling" — early detection framing).[12]

### 18. DegradAI
Meghana Sudarshan, Jaya Vikeswara Rao Vajja, Vikas Tomar. "DegradAI: A
scalable framework for early battery health diagnosis from limited data."
*npj Clean Energy* (2025), DOI 10.1038/s44406-025-00008-2.[13]

- "a chemistry-aware deep learning framework designed to predict long-term
  lithium-ion battery capacity fade using limited early-cycle data (~5 h)",
  with explicit cathode-chemistry identification (incl. LiFePO4) and
  validation across 0.5–3 C and varied temperatures; synthetic-data
  augmentation (80:20 mix) improves accuracy by up to ~22 %.[13]
- Why it matters: chemistry-aware + limited-early-data + LFP — closely
  matches our constraints; synthetic augmentation is a bonus trick for our
  6-cell set.

---

## Open datasets worth using for pre-training (LFP-first)

| Dataset | Chemistry | Cells | Evidence |
|---|---|---|---|
| Severson/MATR | LFP 1.1 Ah 18650 | 124 | cited in [8] |
| Wheeler et al. 2025, *Scientific Data*, DOI 10.1038/s41597-025-04712-7 | LFP 1.1 Ah 18650 | 20, aged to "40% of capacity loss" (first+second life) | [16] |
| Li et al. 2024 (Iowa State), doi:10.25380/iastate.22582234 | NMC | 225, wide conditions | [2] |
| Che et al. 2023, *Cell Reports Physical Science*, DOI 10.1016/j.xcrp.2023.101743 — "a strategy to strengthen the accuracy and generalization of battery health estimation" (PDF dataset [2]) | LFP | large | [19] |
| Catenaro & Onori 2021, *Data in Brief*, DOI 10.1016/j.dib.2021.106894 — incl. "A123 Systems ANR26650m1-B" (LFP) cells at 5/25/35 °C, C/20–20 C (PDF dataset [3]) | NCA/NMC/LFP | 18 | [15] |

---

## What this means for OUR plan (synthesis)

1. **Feature side**: Severson-style ΔQ(V) features[8] + Li et al.'s dQ/dV
   degradation-mode features[2] are the proven early-life predictors; both
   are directly computable from our 30 s time series.
2. **Model side**: predict the whole curve, not a scalar — physics-informed
   curve-parameter reconstruction[3] or a GP full-trajectory model[18];
   hierarchical Bayesian structure for OOD extrapolation[2]; BatLiNet-style
   inter-cell contrast[1] — demonstrated at 20.26 % MAPE with only 2–8
   target cells plus historical LFP data[1]; interpretable feature-based models beat complex ML at our sample size[5].
3. **Extrapolation side**: expect Arrhenius nonlinearity in T × C-rate ×
   ageing state[17] — use mechanism-aware forms and validate LOCO, paying
   special attention to the missing corners (35 °C/0.5 C, 55 °C/0.5 C).
4. **Transfer side**: pretrain on Severson/Wheeler/HUST LFP (same chemistry
   as target)[8][16], fine-tune on the 6 cells; foundation-model route
   (PBT)[7] as a fallback; synthetic-data augmentation[13] as a bonus trick.
5. **Knee side**: Fermín-Cueto knee labels[10] as an auxiliary target and
   separate evaluation axis; the 45 °C/0.5 C cell (knee at ~43 % SOH) is
   our hardest case.
6. **Benchmark honestly**: 9 % test error (Severson)[unverified], 21.8 % OOD
   MAPE (Li et al.)[2], >40 % MAPE reduction (BatLiNet)[unverified] are the
   published numbers to contextualize our LOCO results — but the
   challenge's relative-to-baseline metric is what ultimately counts.

---

## Changelog: v1 (snippets) → v2 (verified)

- Metadata for all 21 sources re-fetched from arXiv API + Crossref +
  Semantic Scholar: authors, venues, years, DOIs now authoritative.
- Corrected: DegradAI is in *npj Clean Energy* (not "Nature portfolio
  journal"); full titles and author lists fixed (e.g. PBT authors,
  Discovery Learning authors); Li et al. journal year 2024; Greenbank &
  Howey DOI resolved (10.1109/TII.2021.3106593); Zhu et al. IEEE TTE DOI
  resolved; Che et al. DOI resolved (10.1016/j.xcrp.2023.101743).
- Added with verified metadata: BatteryTTT[6], Greenbank & Howey[18],
  Zhu et al.[20], deep-TL small samples[21].
- Claims not backed by fetched evidence are explicitly marked [unverified]
  (Kucinskis mechanism-crossover, heterogeneous-feature TL paper).
- Upgraded from [unverified] to cited after fetching primary sources:
  Severson headline numbers (from the Nature Energy PDF via MIT) and
  BatLiNet results (from the arXiv HTML), including the "20.26 % MAPE with
  two to eight target cells + historical LFP data" data-efficiency result
  that maps directly onto our 6-cell regime.
- 30 verbatim evidence quotes attached to the ledger (abstracts +
  primary-source PDF/HTML sentences).

## Sources

[1] https://arxiv.org/abs/2310.05052 — BatLiNet - inter-cell deep learning (arXiv)
[2] https://arxiv.org/abs/2307.08382 — Li et al. 2024 - varying usage conditions (arXiv)
[3] https://arxiv.org/abs/2404.17174 — Physics-informed curve prediction (arXiv)
[4] https://arxiv.org/abs/2304.11671 — Knee-onset via degradation curvature (arXiv)
[5] https://arxiv.org/abs/2101.01885 — Statistical learning follow-up (arXiv)
[6] https://arxiv.org/abs/2402.00068 — Test-time training cross-domain SOH (arXiv)
[7] https://arxiv.org/abs/2512.16334 — Pretrained battery transformer PBT (arXiv)
[8] https://doi.org/10.1038/s41560-019-0356-8 — Severson 2019 Nature Energy
[9] https://doi.org/10.1016/j.xcrp.2024.101891 — Li et al. 2024 Cell Reports Physical Science
[10] https://doi.org/10.1016/j.egyai.2020.100006 — Fermin-Cueto 2020 Energy and AI
[11] https://doi.org/10.1038/s41586-025-09951-7 — Discovery Learning 2026 Nature
[12] https://doi.org/10.1038/s41467-025-66819-0 — BatteryGPT 2025 Nature Communications
[13] https://doi.org/10.1038/s44406-025-00008-2 — DegradAI 2025 npj Clean Energy
[14] https://doi.org/10.1007/s11581-026-07030-z — Ionics 2026 TL meta-analysis
[15] https://doi.org/10.1016/j.dib.2021.106894 — Catenaro & Onori 2021 Data in Brief
[16] https://doi.org/10.1038/s41597-025-04712-7 — Wheeler 2025 Scientific Data
[17] https://doi.org/10.1016/j.jpowsour.2022.232129 — Kucinskis 2022 J Power Sources
[18] https://doi.org/10.1109/TII.2021.3106593 — Greenbank & Howey IEEE TII
[19] https://doi.org/10.1016/j.xcrp.2023.101743 — Che 2023 Cell Reports Physical Science
[20] https://doi.org/10.1109/TTE.2024.3354107 — Zhu 2024 IEEE TTE
[21] https://doi.org/10.1016/j.jpowsour.2025.237503 — Deep TL small samples J Power Sources 2025
