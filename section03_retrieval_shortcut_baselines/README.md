# section 3 — retrieval shortcut baselines (data-distribution problem)

**Goal**: reproduce the two spectrum-blind baselines from MassSpecGym v1.5's pitfalls paper
(`papers/MassSpecGym v1.5 common pitfalls.pdf`, §5.2 / Appendix A.4) — baselines that score
competitively on the retrieval leaderboard while completely ignoring the spectrum, by exploiting the
distributional mismatch between MassSpecGym's ground-truth molecules (predominantly well-studied
metabolites/natural products) and PubChem-derived decoy candidates. This is the checkpoint before
exploring whether the problem can be reframed (property-matched decoys, per Gupta & Skinnider,
`papers/Confronting spurious evaluations...pdf` — read, not reproduced, see `REVIEW.md`).

See `claude/curriculum.md`'s 2026-07-14 update-log entry for how this section's scope was decided,
and `REVIEW.md` for a note on why these two baselines' exact scoring code isn't publicly available
anywhere in the MassSpecGym repo — Appendix A.4's prose is the most precise spec that exists.

## What's different from Section 2

Section 2 trained a real model on a 5-spectrum debug set. This section scores **heuristics** (no
learning, no gradients) against the **real held-out test fold**: 3,170 unique query molecules,
~253 candidates each, 782,695 unique candidate molecules in total. Same `RetrievalMassSpecGymModel`
contract, but `configure_optimizers` returns `None` (precedented by their own `RandomRetrieval`
class) and there's no `.fit()` call at all — only `trainer.test()`.

## Phases

**Phase 0 — load the real test fold**
Build a `RetrievalDataset` against the real v1.5 data (not the debug files), wrapped in a
`MassSpecDataModule`. No `mol_transform` needed — these baselines score raw candidate SMILES
directly (RDKit / PubChem lookups), not a learned fingerprint space.

**Phase 1 — chiral-atom-count baseline (Method 6)**
Fully specified in Appendix A.4: count atoms per candidate with `GetChiralTag() !=
CHI_UNSPECIFIED`, rank by that count. The ranking direction (ascending vs. descending) is decided
empirically from the training set — you need to design how.

**Phase 2 — PubChem default-ranking baseline (Method 5)**
Score candidates by a PubChem "how well-known is this compound" proxy (working interpretation:
Substance-ID count per CID via PUG-REST — see `REVIEW.md`). The PubChem lookup/caching/batching
utility is infrastructure (provided); turning a cached lookup result into a per-candidate score
inside a `step()` is yours.

**Phase 3 — run both via `WandbLogger` + `trainer.test()`**
No `.fit()`. Start with `PILOT_N_QUERIES` capped small (pipeline correctness first, full-scale
782K-candidate run second — cheap to get wrong small, expensive to get wrong at scale).

**Phase 4 — compare against the paper**
Paper's reported Hit rate@1 (with 99.9% bootstrap CI): chiral-atom baseline ~4.88%, PubChem
default-ranking ~49.98%. Close enough to trust the harness, or not, and why.

**Phase 5 — review**
Fold what you learn into `REVIEW.md` — including anywhere your numbers diverge from the paper's and
your best guess why, given Method 5's scoring isn't independently verifiable against real code.

## Prerequisites
- Section 2 complete (`RetrievalMassSpecGymModel` contract, `MassSpecDataModule`, `WandbLogger`
  via the `Trainer`).
- Section 1's RDKit familiarity (Morgan fingerprints) — chiral-atom counting is the same API family.

## Deliverables
- `section03_retrieval_shortcut_baselines.ipynb` — Phases 0-4 running end-to-end on the real test fold.
- `REVIEW.md` updated with Phase 4's comparison and any divergence from the paper's numbers.

## Hard constraints
- No `.fit()` — these are heuristics, not learned models. `configure_optimizers` returns `None`.
- Pilot on `PILOT_N_QUERIES` before running the full test fold.
- Dedupe candidate molecules globally before any PubChem call — don't query per-(spectrum,
  candidate) pair, query per unique candidate.

## Not in scope
- The learned "structure-only classifier" baseline from Gupta & Skinnider — optional stretch, not
  required for this section (see `REVIEW.md`).
- The generative chemical-language-model decoy generator (Gupta & Skinnider's proposed fix) —
  parked as a candidate future direction, not built here.
- DreaMS / MIST as encoders — still Sections 4-5, per the pending curriculum reorder.
