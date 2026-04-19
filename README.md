# metabolomics-ml-curriculum

A public, learning-in-public log from my PhD survey phase at Tel Aviv University (starting April 2026). Supervised by Prof. Yariv Brotman (plant metabolomics, mGWAS).

This repo tracks my twelve-week progression through modern machine learning tools for untargeted metabolomics. Each week is either a **tour** — install a state-of-the-art tool, run it on real data, reproduce a published number, and write a critical review — or a **build** — reimplement something in PyTorch to understand it mechanically.

The curriculum emphasizes depth over breadth in specific places. I skip material such as mzML parsing, cosine similarity, RT alignment, PCA/PLS-DA on feature tables, and invest instead in the modern deep-learning stack that the field has adopted since 2022.

## Weekly structure

- **Weeks 1–2**: PyTorch onboarding; first spectral encoder on a GNPS subset.
- **Weeks 3–6**: Four tour weeks — DreaMS, MIST, MassSpecGym, SIRIUS/CSI:FingerID/CANOPUS. Each with a one-page critical review and an embedded dataset interrogation.
- **Week 7**: Checkpoint, plus the molecular side (ChemBERTa/MolFormer on MoleculeNet).
- **Weeks 8–10**: Build sprint — a shrunken reimplementation of one of the tour models, in PyTorch Lightning with wandb tracking. Lives in its own dedicated repo.
- **Weeks 11–12**: Critical perturbation — ablations and out-of-distribution probes of the build-sprint model.

## Stack

PyTorch · PyTorch Lightning (from Week 4) · Weights & Biases (from Week 2) · HuggingFace `datasets` · RDKit · matchms.

## Repo structure

```
week01_pytorch_basics/
week02_spectral_encoder/
week03_dreams/
...
```

Each weekly folder contains notebooks, a `REVIEW.md` (for tour weeks), and a `REFLECTION.md` (what I built, what I learned, where I was stuck).

## Status

Work in progress. Weeks marked off in [`PROGRESS.md`](PROGRESS.md) as they complete.

## Author

Ezra Schwartz — ezrast@tauex.tau.ac.il
