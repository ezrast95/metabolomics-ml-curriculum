# section 1 — PyTorch basics

**Goal**: Build a full training pipeline from scratch, by hand, on the ESOL dataset (MoleculeNet's aqueous solubility set, ~1128 compounds, loaded from the HuggingFace Hub): a real dataset, but small and easy, so the exercise stays about PyTorch implementation rather than data wrangling. Feel the ceremony (`Dataset` / `DataLoader` / `nn.Module` / training loop / validation loop / checkpointing) so that when Lightning removes it in section 4 you understand what it's abstracting over.

Each molecule's SMILES string becomes a Morgan fingerprint (RDKit); the target is log solubility. This is a **regression** task — MSE loss, RMSE/MAE/R² as metrics.

# Prerequisites
- Familiarity with deep learning basics (I did Andrew Ng ml and deep learning specialization in coursera)
- Learn the Basics module in pytorch docs (https://docs.pytorch.org/tutorials/beginner/basics/intro.html)

## Deliverables
- `section01_pytorch_basics.ipynb` — the pipeline, running end-to-end.

## Hard constraints
- Bare PyTorch only. No Lightning. No wandb.
- HuggingFace `datasets` is allowed for this section, but only to load ESOL — not for any modeling.
- Must include: custom `Dataset`, `DataLoader`, `nn.Module`, optimizer step, validation loop, model checkpoint save/load.

## Not in scope
- Spectra. Metabolomics data enters in section 2.
- Any GPU requirement. Laptop CPU is fine.
- Graph neural nets / learned molecular representations — Morgan fingerprints are a fixed, precomputed feature, not something the model learns. That's section 7 territory.
