# Section 2 — review notes

Covers the Lightning/MassSpecGym architecture work plus the retrival candidate-set investigation.

## Shift from section 1: minimal tools vs. field tools

Section 1 was deliberately minimal — a non-metabolomics HuggingFace dataset (ESOL), hand-rolled
`Dataset`/`DataLoader`/`nn.Module`/training loop, so every step of the PyTorch pipeline had to be felt
before anything got abstracted away. Section 2 flips that: first real contact with how metabolomics MS
annotation is actually analyzed in the field, via MassSpecGym (MSG), which uses PyTorch Lightning under
the hood. The point of this section was **how MSG uses Lightning to structure MS annotation**.

## What MassSpecGym actually standardizes

MSG defines three challenges — de novo structure generation, retrieval, and spectrum simulation — each
with a bonus variant where the molecular formula is given as an additional input (formula is derivable
with high accuracy from MS1 and considered a combintorial problem more then an ML one). Six total task
configurations. ([Bushuiev et al. 2024](https://arxiv.org/abs/2410.23326), §3)

For each challenge, MSG ships: the dataset itself (distributed via HuggingFace), a `Dataset` class that
knows how to construct examples for that challenge, a `LightningDataModule` that wraps it into
train/val/test splits and dataloaders, and standardized metrics that every model gets evaluated on the
same way. The value here is that MSG turns data acquisition, wrangling, and evaluation into a solved, shared problem, so working on the benchmark means working on the model, and results are comparable across the community by construction. That's a sharp contrast with section 1, where I owned every one of those steps myself.

Concretely, the class hierarchy: `MassSpecGymModel` (abstract `pl.LightningModule`) implements
`training_step`/`validation_step`/`test_step` itself and delegates to one method you write, `step(batch,
stage)`. `RetrievalMassSpecGymModel` adds `on_batch_end`, which computes hit_rate@k, MCES@1, and (for
formula-known configs) fingerprint cosine similarity — so a concrete retrieval model only implements
`step()`, returning `{"loss": ..., "scores": ...}`, and gets the entire evaluation suite implented by MSG.

I worked the retrieval challenge as the entry point: `RetrievalDataset` (`SpecBinner` transform on the
spectra, `MolFingerprinter` — Morgan fingerprints — on the structures) wrapped by `MassSpecDataModule`,
against MSG's own 5-spectrum debug set (not the full dataset — the goal was learning the plumbing, not a
real training run). Model: a small feedforward net (`MyFingerprintRetrieval`) subclassing
`RetrievalMassSpecGymModel`, trained via `pl.Trainer` + `WandbLogger` (a different idiom from raw
`wandb.log()` — you call `self.log(...)` inside `step()`, Lightning forwards it to the logger through
the `Trainer`).

## Insight: retrieval splits into two separable components

Implementing the model surfaced something not obvious from the task description alone: "retrieval"
factors into (1) a loss function that trains the model to predict something about the candidate
molecule (here, a fingerprint, scored against the model's prediction by cosine similarity) and (2) the
ranking/search step that turns those scores into an ordering over the candidate set. These don't have to
be coupled — a model can be trained to directly optimize ranking quality (learned-to-rank) instead of
fingerprint-prediction accuracy, and MSG's own scaffolding doesn't force one over the other; it just asks
`step()` to return `scores`.

This is a real, current tension in the field, not just a design footnote: [De Waele et al.,
"Small molecule retrieval from tandem mass spectrometry: what are we optimizing for?" (Feb 2026, arXiv:2602.16507)](https://arxiv.org/abs/2602.16507)
shows a theoretical and empirical trade-off between optimizing for fingerprint-prediction accuracy and
optimizing for retrieval accuracy directly — improving one typically worsens the other, and which one
wins depends on the similarity structure of the candidate set. Worth returning to when picking a loss
for any retrieval model going forward, not just as a footnote.

Concrete evidence this isn't just theoretical: [Krzakala et al., "MSAlign: Aligning Molecule and Mass
Spectra Foundation Models for Metabolite Identification" (May 2026, arXiv:2605.19752)](https://arxiv.org/abs/2605.19752),
Table 4, shows on MassSpecGym (formula split) that a candidate-based contrastive loss (directly
optimizing ranking) reaches 53.8% R@1 versus 24-27% R@1 for regression-based fingerprint prediction
(predict-then-search) — the same distinction as above, with a ~2x gap between the two approaches.

## Insight: optimizer inconsistency across MSG's own model base classes

`MassSpecGymModel.configure_optimizers` (the base every retrieval/de-novo model inherits) hardcodes
`torch.optim.Adam` with a `weight_decay` hyperparameter (default `0.0`) —
`massspecgym/models/base.py:99-103`. `SimulationMassSpecGymModel.configure_optimizers`
(`massspecgym/models/simulation/base.py:87-101`) is the only place in the codebase that exposes an
`optimizer_type` choice between `"adam"`, `"adamw"`, and `"sgd"`.

Adam's `weight_decay` applies L2 regularization inside the adaptive-learning-rate update, which doesn't
behave like true weight decay; `AdamW` decouples the two and is the standard recommendation whenever
`weight_decay != 0` ([Loshchilov & Hutter, "Decoupled Weight Decay Regularization," ICLR 2019](https://arxiv.org/abs/1711.05101)).
Not a bug — the default `weight_decay=0.0` means it's inert unless a model config actually sets it — but
it is an inconsistency: the fix (giving `optimizer_type` a real choice) exists in MSG's own codebase for
the simulation base class and simply wasn't lifted into the shared base that retrieval and de novo
models inherit from. Weirth remembering an implementing AdemW in retrieval and de novo models if weight decay is needed. 

## Insight: the retrieval candidate sets carry a PubChem popularity shortcut

While reading `RetrievalDataset`'s candidate-set construction to understand the retrieval data pipeline
(not something the section 2 plan called for — fell out of wanting to understand what a "candidate" in
`step()`'s `scores` output actually was), found a chain of issues worth carrying forward into any future
retrieval work on this benchmark.

**How a candidate set is supposed to be built.** MSG samples decoys with the same mass as the ground
truth from three molecule pools, in priority order, until the 256-candidate cap is reached: (i) ~1M
molecules of biological/environmental origin — natural products, pesticides, industrial chemicals, food
additives (Small molecule machine learning: All models are wrong, some may not even be useful (https://www.biorxiv.org/content/10.1101/2023.03.27.534311v1)); (ii) ~4M molecules across diverse chemical classes
(Systematic classification of unknown metabolites using high-resolution fragmentation mass spectra); (iii) all 118M molecules from PubChem. The paper states explicitly that "the sequence of the datasets used for sampling reflects the relevance of
their composition for mass spectrometry applications" — i.e., (i) and (ii) are meant to dominate,
with PubChem as the last resort.

**What the actual candidate-generation notebook does.** MSG's own
`notebooks/retrieval_candidates_construction/3_generate_retrieval_candidates.ipynb` searches all three
pools but only keeps the PubChem search results for the final candidate set — the (i)/(ii) searches run
and get discarded. That's the opposite of what the paper's own text describes as the intended priority.
Mechanically: the ground-truth SMILES is injected into the candidate list directly, decoys are pulled by
matching mass (10 ppm) or formula (depending on the challange) and excluding shared 2D InChIKey, then the set is shuffled and capped at 256.
(Caveat carried over from when this was investigated: that notebook's file paths are stale against the
current repo layout, so it's unclear whether it's still an accurate record of how the released candidate
files were actually produced — not resolved further.)

**Why this creates a shortcut.** The ground truth molecule had to pass a real-world filter the decoys
never did — it was physically synthesized/isolated and actually measured to produce a reference
spectrum. PubChem-sourced decoys only had to match its mass; nothing about how "well-known" a compound
is factors into which ones get pulled in. So the ground truth is systematically more likely to be a
well-known/well-documented compound than an arbitrary same-mass decoy — meaning a ranker that scores
candidates by "how well-known is this" (no spectrum needed) correlates with correctness for reasons that
have nothing to do with mass spectrometry.

**Confirmed evidence this isn't hypothetical.** MSG's own v1.5 pitfalls paper (`papers/MassSpecGym v1.5
common pitfalls.pdf`, §5.2) reports a spectrum-blind "rank by PubChem deposition count (how many unique reference sources does this compound have in pubchem)" baseline reaching
~50% Hit@1 — in the same range as the strongest real models (Table 3). The paper flags this as an open
problem and proposes a check (mask the input spectrum, confirm performance degrades) but states this
check was **not** applied to any of the leaderboard's real competing methods — so the leaderboard
numbers can't currently be read as proof those methods aren't partly riding the same shortcut. Separately
verified against the shipped candidate files (not just the notebook): the ground-truth molecule sits at
index 0 of its own candidate list in both the mass- and formula-based
files, because it's appended before the decoy search runs and only the decoys get shuffled afterward —
a second, more basic shortcut the papers don't discuss, exploitable only by architectures that treat
candidate order as meaningful.

**Follow-up, deliberately deferred, not forgotten.** Wanted to reproduce the two spectrum-blind
baselines from the v1.5 paper's Appendix A.4 (chiral-atom-count ranking via RDKit's `GetChiralTag`, and
the PubChem-popularity ranking) directly, but running them meaningfully requires comparing
against real trained retrieval models, not just against each other — and training a real model on
anything beyond the 5-spectrum debug set isn't practical on this laptop. The actual point of this
ablation only pays off once there's a real model to test it against. **When a real retrieval model
exists:** run the model with test data spectrum-blind and compare to baseline, and separately test performance with candidate order shuffled (since ground truth always starts
at index 0) — both are cheap sanity checks for whether a trained model is exploiting these shortcuts
rather than reading the spectrum.

## Practical note: which MassSpecGym to actually install

MSG exists in three places — PyPI, GitHub, and HuggingFace (data/models) — and in two versions, v1 and
v1.5. The v1.5 code (with the fixes the pitfalls paper discusses) was never published to PyPI (latest
PyPI release, `1.3.1`, predates it); it only exists as the `feat/massspecgym-v1.5` branch merge on
GitHub. Pinned `pyproject.toml` directly to that merge commit
(`f259fe3780d5bd227fc6ece36ce6f397c2eef716`, 2026-05-08) rather than a branch name, since branches move
and this needs to stay reproducible.

## Open threads

- Loss-function-vs-ranking-objective tension (De Waele et al.) — revisit before choosing a loss for any
  future retrieval model, not just noting it here.
- The `optimizer_type` inconsistency is a one-line MSG upstream issue worth filing at some point, not
  urgent.
- Candidate-pool provenance (does the released data actually follow the (i)-(ii)-(iii) priority order
  the paper describes, or PubChem-only as the notebook suggests) is still genuinely unresolved — would
  need MSG maintainer input, not something resolvable from this repo alone.
- Ask MSG maintainers how the three source pools (1M/4M/118M) were themselves built from the cited
  papers — no construction code is public.
- Shortcut ablations (PubChem deposition count, shuffled candidate order)
  — parked until a real trained model exists to run them against - when ready then run the model with blind spectrum, suffle candidates to make sure models degrats to baseline preformance. 

## Possible directions (high-level, unevaluated)

1. Fix MSG's candidate generation to actually follow the paper's stated (i)→(ii)→(iii) priority instead
   of the PubChem-only behavior the notebook currently has.
2. Switch to Spectraverse-style decoys instead of MSG's mass/formula-matched PubChem pull:
   [Confronting spurious evaluations of computational methods in small molecule mass spectrometry (bioRxiv, May 2026)](https://www.biorxiv.org/content/10.64898/2026.05.03.722532v1)
   generates decoys with a model trained only on SMILES of compounds that have reference spectra, so
   decoys are chemically indistinguishable from the ground truth on "is this the kind of thing that gets
   measured" — closing the exact gap this section's shortcut exploits.
3. Instead of fixing the data, design tests/metrics/probes that detect whether a *given trained model* is
   exploiting the shortcut (masked-spectrum, shuffled-candidate — already noted above) rather than
   preventing it at the dataset level.
4. Restrict the candidate pool to just the ~1M biological/environmental compound set and drop PubChem
   entirely — arguably the more research-relevant question anyway, since that's the compound space
   practitioners actually care about identifying.
5. Apply MSAlign's normalized-Wasserstein train/test distribution-shift metric directly to
   ground-truth-vs-decoy distributions within a candidate set, to turn "the shortcut looks suspicious"
   into an actual measured number instead of an inference from Hit@1 alone.
