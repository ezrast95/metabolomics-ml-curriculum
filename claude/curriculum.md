# Coding Curriculum — ML for Metabolomics

Progressive, hands-on plan for a PhD in ML for metabolomics.

## Goals of this curriculum
Two things in parallel. First, become fluent with the modern ML tooling that dominates metabolomics today — DreaMS, MIST, MassSpecGym, SIRIUS/CSI:FingerID/CANOPUS — to the point of being able to run them, criticize them, and extend them. Second, build a public GitHub portfolio that signals to industry data-science and bioinformatics hiring managers that the modern stack is in hand. The end state is one polished portfolio-grade project and a clear sense of where I might contribute in the field.

## Budget
One hour per day, roughly seven hours per section. Twelve sections total. No laboratory cluster dependency in the first ten sections. A Windows laptop CPU plus occasional free Colab GPU session is enough through Section 10.

## Stack
PyTorch (not TensorFlow — TF is effectively dead in academic chem/bio ML). PyTorch Lightning introduced in Section 2 (moved up from Section 4 — see update log) via MassSpecGym's own `LightningModule` scaffolding, rather than as a standalone abstraction. Weights & Biases for experiment tracking from Section 2. HuggingFace `datasets` for streaming large spectral datasets. RDKit for molecular work. matchms for spectrum IO when necessary. Git and GitHub from Day 1, two-tier structure: one public `metabolomics-ml-curriculum` repo for all sectionly tour and exercise work, and a separate dedicated public repo for the Sections 8-10 portfolio centerpiece.

## Database criticism — embedded, not a separate section
Every tour section has a 30-60 minute block where the question is not "does the model work" but "what was it trained on." For each model I write down: source libraries pooled, filters applied, positive/negative mode split, collision-energy distribution, taxonomic coverage (plant vs microbial vs human), how train/val/test were split (by compound? by scaffold? by dataset?), and licensing. By Section 6 this becomes habit rather than a task.

## Section 1 — PyTorch onboarding and a training loop from scratch
PyTorch's official "Learn the Basics" tutorial notebooks are done — the mechanics have been read, now they need to be typed. Practice by implementing the full pipeline from scratch, by hand, on the ESOL dataset (MoleculeNet's aqueous solubility set, ~1128 compounds, pulled from HuggingFace): a real dataset, but small and easy, chosen so the exercise stays about PyTorch ceremony, not about wrestling with data. Each molecule's SMILES string gets turned into a Morgan fingerprint (RDKit, fixed-length bit vector) as model input; the target is log solubility — a regression task, so MSE loss and RMSE/MAE/R² as metrics, a different flavor than Section 2's classifier. Write `Dataset`, `DataLoader`, a two-to-three-layer `nn.Module`, a training loop, a validation loop, and checkpointing. No Lightning, no wandb — the point is to feel the ceremony Lightning removes before Lightning removes it. Commit to `metabolomics-ml-curriculum` as `section01_pytorch_basics/`.

## Section 2 — PyTorch Lightning, through MassSpecGym's own architecture
Revised from the original from-scratch-GNPS-encoder plan (see update log below). Learn PyTorch Lightning specifically as MassSpecGym's own codebase uses it: their `MassSpecGymModel` -> `RetrievalMassSpecGymModel` -> concrete-model class hierarchy, practiced by implementing a small feedforward retrieval model against their real `RetrievalDataset`/`MassSpecDataModule` on their own 5-spectrum debug set. Weights & Biases via `WandbLogger` through the `Trainer`, not raw `wandb.log()`. Full plan in `section02_lightning_massspecgym/README.md`.

## Section 3 — Tour: DreaMS
Install DreaMS. Download their pretrained model and their public dataset release on HuggingFace. Run their spectrum retrieval benchmark on the held-out set and reproduce one reported number. Interrogate the dataset as described above. One-page critical review in `section03_dreams/REVIEW.md`: what DreaMS does well, where it will fail, which assumptions are brittle.

Infrastructure already set up ahead of time (Section 2, 2026-07-06): `envs/dreams-env/`, an isolated environment with DreaMS installed and verified working (`dreams.api.dreams_embeddings` tested end-to-end, checkpoints downloaded) — DreaMS's pinned dependencies directly conflict with `massspecgym`'s (torch, pytorch-lightning, numpy, rdkit, wandb, matchms, huggingface-hub all pinned to different exact versions), so it can't live in the main environment. There was also a parked idea worth revisiting here: using DreaMS (and optionally MIST) as frozen encoders on MassSpecGym data, visualizing embeddings with UMAP following DreaMS's own tutorial pattern, and probing what the embedding space captures (modeled on their DreaMS-Fluorine tutorial). Decide when reaching this section whether that folds in here.

## Section 4 — Tour: MIST
Install MIST (Goldman et al., contrastive spectrum-to-structure) in its own isolated environment (same reason as DreaMS in Section 3 — MIST's pinned dependencies are old research-code versions, unlikely to coexist with `massspecgym`'s environment). Run inference on a small test set and reproduce one reported number. Same dataset interrogation as Section 3. The Lightning migration originally planned for this section already happened in Section 2 — this section's scope needs a fresh look when reached, since a chunk of what Sections 3-5 were meant to cover (using DreaMS/MIST as pretrained encoders, embedding visualization/comparison via UMAP) may fold in here or in Section 3 instead. The MIST review goes in `section04_mist/REVIEW.md`.

## Section 5 — Tour: MassSpecGym, the field's current benchmark
The most important tour section. MassSpecGym (NeurIPS 2024) defines the tasks most groups are now benchmarking on: de novo structure generation from MS2, retrieval, Bayesian retrieval, and more. Install, run the baselines, reproduce one result, read the paper in depth. The review focuses specifically on which of their tasks looks most tractable for a PhD-sized contribution, and which splits are strict enough that SOTA papers still struggle. This document is the single most useful artifact for PhD-topic scouting.

## Section 6 — Tour: SIRIUS + CSI:FingerID + CANOPUS
The incumbent non-deep-learning system that every modern tool implicitly compares itself to. Install, run on a small real dataset from MetaboLights, reproduce one annotation. Review focus: how do the deep-learning tools from Sections 3-5 compare to SIRIUS on real data, not just on curated benchmarks. This is where the intuition forms that separates PhD students who parrot benchmark numbers from those who understand the field.

## Section 7 — Pause, reflect, and cross to the molecular side
Two goals. First, pick the one or two tours from Sections 3-6 that were most interesting — these seed the build sprint. Second, spend this section's coding on the SMILES/molecule side, because this is the bridge to Pupko. Fine-tune a small ChemBERTa or MolFormer checkpoint from HuggingFace on a simple property task — solubility or toxicity from MoleculeNet. Commit as `section07_molecular/`. Write a short reflection in `CURRICULUM_CHECKPOINT.md` with the chosen build direction and why.

## Sections 8-10 — Build sprint
Reimplement a shrunken version of the model chosen in Section 7, from scratch in PyTorch Lightning with wandb tracking. Dedicated public repo, polished README, reproduction instructions, a plot or two. Try to match — or get close to — a published baseline on a small subset of the relevant benchmark. Scope ruthlessly: "shrunken" means fewer layers, smaller dataset, shorter training. It does not mean reduced implementation quality. This repo will be the top pinned item on my profile.

## Sections 11-12 — Critical perturbation
The graduation move. Take the model from Sections 8-10 and probe its weaknesses: ablate a component and measure the hit, hold out a compound class to test out-of-distribution generalization, swap in a different tokenizer or loss. Write findings in the same repo as an `ABLATIONS.md`. This is the transition from "I can use modern ML" to "I can identify where modern ML breaks" — the actual skill PhD-topic selection requires.

## Checkpoint: cluster becomes non-optional
At the end of Section 10, regardless of eventual PhD topic, I need to resolve TAU cluster access. Even if I rarely use it, at least a few real GPU runs will be required for any serious reproduction. First test job submitted by end of Section 12.

## Parallel track
Andrew Ng's Deep Learning Specialization runs alongside the sectionly curriculum, decoupled from it. No blocking dependency — the PyTorch onboarding in Section 1 covers what's needed to start, and DLS content will sharpen understanding as it lands.

## Sectionly reflection template
Each section, a `REFLECTION.md` in the section's folder with three short sections: what I built, what I learned, and where I was stuck and for how long. These accumulate into a learning-in-public narrative that becomes part of the portfolio.

---

## Progress

| Section | Topic | Status | Completed |
|------|-------|--------|-----------|
| 1    | PyTorch onboarding | Complete | 2026-07-05 |
| 2    | PyTorch Lightning via MassSpecGym's architecture | Not started | — |
| 3    | Tour: DreaMS | Not started | — |
| 4    | Tour: MIST + Lightning migration | Not started | — |
| 5    | Tour: MassSpecGym | Not started | — |
| 6    | Tour: SIRIUS + CSI:FingerID + CANOPUS | Not started | — |
| 7    | Checkpoint + molecular (ChemBERTa/MolFormer) | Not started | — |
| 8-10 | Build sprint (dedicated repo) | Not started | — |
| 11-12| Critical perturbation | Not started | — |

Update this table as sections complete. Keep short dated notes below for anything a status cell can't capture (slippage, scope changes, blockers).

### Update log
_Section 1 (2026-07-05)_: Complete. MLP regression on ESOL dataset. Full pipeline from scratch: Dataset, DataLoader, nn.Module, training loop, validation loop, checkpoint, loss curve visualisation. Gaps: loss/metric variety not yet solid, HuggingFace datasets only first contact.

_Section 2 scope change (2026-07-06)_: Reoriented the curriculum around MassSpecGym starting now instead of Section 5. Reasoning: the actual PhD-relevant goal is to perform well on the MassSpecGym benchmark using the field's existing architecture (not reinvent spectrum encoding from scratch), and MassSpecGym's own codebase is built on PyTorch Lightning — so Lightning fluency, learned directly through their `LightningModule` hierarchy, is more valuable now than a from-scratch GNPS encoder. Went through several intermediate ideas first (from-scratch encoder on GNPS binned-vs-continuous comparison; DreaMS/MIST as frozen encoders with UMAP visualization) before landing here — those aren't wasted: `envs/dreams-env/` is set up and verified for whenever DreaMS resurfaces, and the embedding-comparison idea is parked for Section 3 or 4. Consequence: Section 4's originally-planned Lightning migration already happened; Sections 3-5 need their scope re-examined when reached, since some of what they covered (Lightning intro, first hands-on MassSpecGym contact) is now done early. Environment note: `massspecgym`'s pinned dependencies (torch==2.3.0, Python>=3.11) forced bumping this repo's Python from 3.10 to 3.11 and relaxing version floors that were set in Section 1 — no functional impact expected on Section 1's completed work (it only uses basic, stable PyTorch APIs), but worth knowing if anything there ever needs revisiting.
