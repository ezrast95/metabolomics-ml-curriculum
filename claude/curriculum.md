# Coding Curriculum — ML for Metabolomics

Progressive, hands-on plan for a PhD in ML for metabolomics.

## Goals of this curriculum
Two things in parallel. First, become fluent with the modern ML tooling that dominates metabolomics today — DreaMS, MIST, MassSpecGym, SIRIUS/CSI:FingerID/CANOPUS — to the point of being able to run them, criticize them, and extend them. Second, build a public GitHub portfolio that signals to industry data-science and bioinformatics hiring managers that the modern stack is in hand. The end state is one polished portfolio-grade project and a clear sense of where I might contribute in the field.

## Budget
One hour per day, roughly seven hours per section. Twelve sections total. No laboratory cluster dependency in the first ten sections. A Windows laptop CPU plus occasional free Colab GPU session is enough through Section 10.

## Stack
PyTorch (not TensorFlow — TF is effectively dead in academic chem/bio ML). PyTorch Lightning introduced in Section 4 after feeling the pain Lightning removes. Weights & Biases for experiment tracking from Section 2. HuggingFace `datasets` for streaming large spectral datasets. RDKit for molecular work. matchms for spectrum IO when necessary. Git and GitHub from Day 1, two-tier structure: one public `metabolomics-ml-curriculum` repo for all sectionly tour and exercise work, and a separate dedicated public repo for the Sections 8-10 portfolio centerpiece.

## Database criticism — embedded, not a separate section
Every tour section has a 30-60 minute block where the question is not "does the model work" but "what was it trained on." For each model I write down: source libraries pooled, filters applied, positive/negative mode split, collision-energy distribution, taxonomic coverage (plant vs microbial vs human), how train/val/test were split (by compound? by scaffold? by dataset?), and licensing. By Section 6 this becomes habit rather than a task.

## Section 1 — PyTorch onboarding and a training loop from scratch
PyTorch's official "Learn the Basics" tutorial notebooks are done — the mechanics have been read, now they need to be typed. Practice by implementing the full pipeline from scratch, by hand, on the ESOL dataset (MoleculeNet's aqueous solubility set, ~1128 compounds, pulled from HuggingFace): a real dataset, but small and easy, chosen so the exercise stays about PyTorch ceremony, not about wrestling with data. Each molecule's SMILES string gets turned into a Morgan fingerprint (RDKit, fixed-length bit vector) as model input; the target is log solubility — a regression task, so MSE loss and RMSE/MAE/R² as metrics, a different flavor than Section 2's classifier. Write `Dataset`, `DataLoader`, a two-to-three-layer `nn.Module`, a training loop, a validation loop, and checkpointing. No Lightning, no wandb — the point is to feel the ceremony Lightning removes before Lightning removes it. Commit to `metabolomics-ml-curriculum` as `section01_pytorch_basics/`.

## Section 2 — First spectral encoder on GNPS
Download a GNPS library subset (or reuse matchms's tutorial subset). Build a minimal MS2 spectrum encoder in PyTorch: peaks as tokens, m/z as continuous positional information, MLP or tiny transformer head. Train on a trivial binary task — for example "plant-derived vs microbial-derived" using taxonomy metadata. The task is scientifically meaningless; the goal is a working end-to-end spectrum-in, label-out pipeline. Add Weights & Biases logging this section. Reflection in the notebook: what broke, what was learned about spectrum tokenization.

## Section 3 — Tour: DreaMS
Install DreaMS. Download their pretrained model and their public dataset release on HuggingFace. Run their spectrum retrieval benchmark on the held-out set and reproduce one reported number. Interrogate the dataset as described above. One-page critical review in `section03_dreams/REVIEW.md`: what DreaMS does well, where it will fail, which assumptions are brittle.

## Section 4 — Tour: MIST, and migrating to PyTorch Lightning
Install MIST (Goldman et al., contrastive spectrum-to-structure). Run inference on a small test set and reproduce one reported number. Same dataset interrogation as Section 3. This is also the section my own code graduates into Lightning — refactor Section 2's training loop into a `LightningModule` with proper callbacks. The MIST review goes in `section04_mist/REVIEW.md`.

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
| 2    | First spectral encoder on GNPS | Not started | — |
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
