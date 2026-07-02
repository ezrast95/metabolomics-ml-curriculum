# Week 1 — PyTorch basics

**Goal**: PyTorch's official "Learn the Basics" tutorials are done — the mechanics have been read, now they need to be typed. Build a full training pipeline from scratch, by hand, on the ESOL dataset (MoleculeNet's aqueous solubility set, ~1128 compounds, loaded from the HuggingFace Hub): a real dataset, but small and easy, so the exercise stays about PyTorch ceremony rather than data wrangling. Feel the ceremony (`Dataset` / `DataLoader` / `nn.Module` / training loop / validation loop / checkpointing) so that when Lightning removes it in Week 4 I understand what it's abstracting over.

Each molecule's SMILES string becomes a Morgan fingerprint (RDKit); the target is log solubility. This is a **regression** task — MSE loss, RMSE/MAE/R² as metrics — a different flavor than Week 2's classifier.

## Deliverables
- `week01_pytorch_basics.ipynb` — the pipeline, running end-to-end.
- `REFLECTION.md` — three short sections: what I built, what I learned, where I got stuck.

## Hard constraints
- Bare PyTorch only. No Lightning. No wandb.
- HuggingFace `datasets` is allowed for this week, but only to load ESOL — not for any modeling.
- Must include: custom `Dataset`, `DataLoader`, `nn.Module`, optimizer step, validation loop, model checkpoint save/load.

## Not in scope
- Spectra. Metabolomics data enters in Week 2.
- Any GPU requirement. Laptop CPU is fine.
- Graph neural nets / learned molecular representations — Morgan fingerprints are a fixed, precomputed feature, not something the model learns. That's Week 7 territory.
