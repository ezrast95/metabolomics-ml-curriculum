# section 2 — first spectral encoder on GNPS

**Goal**: build a minimal MS2 spectrum encoder in PyTorch and get comfortable with `matchms`, the library used throughout the field for spectrum IO and processing. This section is also where the architecture question for sections 3-4 gets asked directly: DreaMS (section 3) and MIST (section 4) are both, at their core, spectrum encoders that turn a variable-length, permutation-invariant set of peaks into a fixed representation. Section 2 practices that core move twice, at two levels of sophistication, on the same task and dataset.

## Open question to answer in the notebook, after Phase 0

What should the classification/regression target be? This is deliberately not decided here — it should be decided from what Phase 0's interrogation actually shows about the data (class sizes, balance, ion-mode/adduct diversity), not guessed at beforehand. Candidates to weigh, once the real numbers are in front of you:

- **Adduct-type classification** ([M+H]+ / [M+Na]+ / [M-H]- / etc.). Real annotation task, label often has to be parsed from the compound name string rather than a clean metadata field, and is frequently thin/imbalanced in practice — checked empirically before committing.
- **Precursor m/z regression** (predict the exact precursor mass from the fragment peaks, precursor peak excluded from input). Every spectrum has this field, so no class-imbalance or missing-label risk; resolution loss shows up as a computable quantization-error floor as bin width grows.
- **Ion-mode classification** (positive vs negative). Only viable if the chosen data actually contains both modes — some GNPS library exports are single-mode.
- Anything else Phase 0 surfaces as a genuinely well-populated, resolution-sensitive field.

Do **not** use plant-vs-microbial / taxonomy origin — there's no reason fine mass resolution should matter for it, so it wouldn't show the thing this section exists to demonstrate.

## Phases

**Phase 0 — GNPS dataset interrogation via `matchms`**
`matchms` does not itself bundle a training-sized GNPS dataset — its own test fixtures are unit-test scale (e.g. `pesticides.mgf`: 76 spectra, single ion mode, adduct classes as imbalanced as 53/13/8/1/1; `gnps_spectra.json`: 5 spectra, single adduct). Neither is usable for training. Pull an actual GNPS spectral library export instead (bulk MGF download from GNPS's library page), then load and harmonize it with `matchms` (`load_from_mgf`, `default_filters`, metadata inspection) — this still delivers real fluency with the API, just not from matchms's toy test data.

Practice the database-criticism habit one section early: what's in this library (source, positive/negative ion mode split, taxonomy coverage, collision energy distribution, and — directly relevant to the open question above — adduct/ion-mode class sizes and balance), since GNPS is one of the three libraries MassSpecGym (section 5) pools its own dataset from. This interrogation is what the open question above gets answered from.

Writing your own curation/filtering pipeline over multiple raw GNPS deposits (rather than one existing library export) is worth doing eventually — parked as a stretch goal, likely to resurface in section 5 when pulling MassSpecGym's own source data.

**Phase 1 — binned-spectrum MLP baseline**
Bin each spectrum's peaks into fixed-width m/z bins (intensity per bin), reusing section 1's `Dataset` / `DataLoader` / `nn.Module` / training-loop / validation-loop patterns. This is the fast, known-quantity path: gets the pipeline (data -> model -> loss -> metrics) working end to end before touching anything novel. Also the resolution-loss tradeoff to feel directly: what does binning throw away.

**Phase 2 — continuous peak encoding + tiny transformer encoder**
Same task, same dataset. Peaks as a set of `(m/z, intensity)` pairs, no binning — m/z encoded as a continuous value (not a discrete vocabulary token) via a from-scratch positional-encoding scheme, fed through a small self-attention encoder. This is the first from-scratch attention implementation of the curriculum, and the direct rehearsal for DreaMS's peak-tokenization approach. Compare against the Phase 1 baseline on the same held-out split.

**Wandb** — introduced this section per the curriculum; log both phases' training runs.

## Prerequisites
- Section 1 complete (`Dataset` / `DataLoader` / `nn.Module` / training loop / checkpointing).
- Familiarity with the Q/K/V attention picture from Ng's Deep Learning Specialization (conceptual only — Phase 2 is the first from-scratch implementation).

## Deliverables
- `section02_spectral_encoder_gnps.ipynb` — all three phases, running end-to-end.

## Hard constraints
- Bare PyTorch only. No Lightning (introduced section 4).
- `matchms` for all spectrum IO/filtering — no hand-rolled MGF parsing.
- Must include: Phase 0 interrogation notes, Phase 1 binned MLP, Phase 2 continuous-encoding transformer, Weights & Biases logging for both trained models.

## Not in scope
- Writing your own multi-source curation/filtering pipeline from raw GNPS deposits (as opposed to downloading one existing library export) — parked as a stretch goal.
- Reproducing DreaMS or MIST themselves — that's sections 3-4. This section only rehearses the underlying representation problem.
- Structure/fingerprint prediction, retrieval, or any MassSpecGym task — that's section 5.
