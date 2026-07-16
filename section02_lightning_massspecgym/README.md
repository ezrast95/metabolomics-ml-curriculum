# section 2 — PyTorch Lightning, through MassSpecGym's own architecture

**Goal**: learn PyTorch Lightning not as a generic abstraction, but specifically the way MassSpecGym
actually uses it — because MassSpecGym is the benchmark this whole curriculum is now oriented around
(clear metrics to improve on, and a large body of existing architecture worth being fluent in rather than
reinventing). The fastest, thoroughest route to being able to implement new models against their benchmark
is to understand the `LightningModule`/`LightningDataModule` contract their codebase is built on, using
their own data pipeline from the start.

This replaces the originally-planned "build a spectral encoder from scratch on GNPS" — after a lot of
back and forth, the decision was to center the curriculum on MassSpecGym starting now rather than in
section 5, and to lean on the field's existing tooling (matching the curriculum's own stated goal: become
fluent enough to run, criticize, and extend existing tools, not reinvent them). See `claude/curriculum.md`'s
update log for the full reasoning trail.

## What MassSpecGym's own architecture looks like (verified against their actual source)

`massspecgym.models.base.MassSpecGymModel` is an abstract `pl.LightningModule` subclass every one of
their models inherits from. It does **not** ask you to override `training_step`/`validation_step`/
`test_step` directly (the standard vanilla-Lightning tutorial pattern) — instead it implements all three
itself, each delegating to a single abstract method you write: `step(self, batch, stage)`. It also
provides `configure_optimizers` (plain Adam) for free.

One layer down, `massspecgym.models.retrieval.base.RetrievalMassSpecGymModel` implements the other
abstract piece (`on_batch_end`, which computes hit_rate@k, MCES@1, and optionally fingerprint cosine
similarity) so that any concrete retrieval model only has to implement `step()` — returning
`{"loss": ..., "scores": ...}` — and gets the whole evaluation suite for free.

Read `massspecgym/models/base.py` and `massspecgym/models/retrieval/base.py` directly (installed in
`.venv/Lib/site-packages/massspecgym/`) before starting — they're short, and this section is about
understanding that contract, not about the notebook explaining it for you.

## MassSpecGym version in use

`massspecgym` is pinned in `pyproject.toml` (`[tool.uv.sources]`) to a specific GitHub commit, not the
PyPI release:

```
git+https://github.com/pluskal-lab/MassSpecGym.git@f259fe3780d5bd227fc6ece36ce6f397c2eef716
```

That commit is the merge of `feat/massspecgym-v1.5` (2026-05-08) — the code release accompanying the
"MassSpecGym in the Wild" pitfalls paper (`papers/MassSpecGym v1.5 common pitfalls.pdf`). It was never
published to PyPI (latest PyPI release is `1.3.1`, from March 2025, well before this work), so a pinned
commit is the only way to get it reproducibly.

## Phases

**Phase 0 — orientation**
Read the two base-class files above. Load MassSpecGym's own small debug set (5 spectra — real data, from
their repo, at `data/massspecgym_debug/`) through their real `RetrievalDataset` + `MassSpecDataModule`,
with their `SpecBinner` / `MolFingerprinter` transforms. Inspect what a batch actually looks like.

**Phase 1 — implement a retrieval model against their scaffolding**
Subclass `RetrievalMassSpecGymModel` yourself: a small feedforward network that takes a binned spectrum
and predicts a molecular fingerprint, scored against candidates by cosine similarity. This is deliberately
at the same complexity as their own `FingerprintFFNRetrieval` (don't read that file until after you've
implemented yours — compare afterward, not before).

**Phase 2 — train it with their `Trainer` + `WandbLogger`**
Run `trainer.validate` / `.fit` / `.test` against the debug data. Note the real gotcha their own demo
notebook flags: `trainer.validate()` before any `.fit()` call needs `data_module.prepare_data()` and
`.setup()` called manually first. Wire up `WandbLogger` — this is a different idiom from raw
`wandb.init()`/`wandb.log()`: you call `self.log(...)` inside `step()` and Lightning forwards it.

**Phase 3 (stretch, only if time) — compare against their own baselines**
Run their pre-built `RandomRetrieval` and `FingerprintFFNRetrieval` on the same debug data and compare
metrics against your own implementation, as a sanity check that your model is behaving sensibly.

## Prerequisites
- Section 1 complete (comfortable with plain PyTorch: `Dataset`/`DataLoader`/`nn.Module`/training loop).
- No prior Lightning experience assumed — that's the point of this section.

## Deliverables
- `section02_lightning_massspecgym.ipynb` — Phases 0-2 running end-to-end on the debug data.

## Hard constraints
- Use MassSpecGym's own `RetrievalDataset`, `MassSpecDataModule`, and transforms — no hand-rolled
  `Dataset`/`DataLoader` this section. The point is their data pipeline, not rebuilding one.
- Your model must subclass `RetrievalMassSpecGymModel`, not raw `pl.LightningModule` — that's the actual
  contract worth practicing.
- `WandbLogger` via the `Trainer`, not raw `wandb.log()` calls.

## Not in scope
- DreaMS or MIST as encoders — that idea (embedding visualization/comparison via UMAP) is parked, not
  discarded; it fits naturally as a later tour section once the Lightning/MassSpecGym foundation is solid.
  `envs/dreams-env/` (an isolated environment for DreaMS, since its pinned dependencies are incompatible
  with `massspecgym`'s) is already set up and verified working for whenever that resurfaces.
- Training on the full MassSpecGym dataset — the 5-spectra debug set is enough to learn the plumbing.
  Real-scale training is for the eventual build sprint.
- De novo generation / spectrum simulation tasks, or the `DeepSets`/`SmilesTransformer` architectures —
  retrieval with a plain FFN is the entry point; the rest of their model zoo is there to explore once this
  contract is second nature.
