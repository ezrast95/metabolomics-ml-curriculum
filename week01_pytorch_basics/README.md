# Week 1 — PyTorch basics

**Goal**: rewrite one of Andrew Ng's TensorFlow neural-network assignments in bare PyTorch, with no Lightning and no wandb. Feel the ceremony (Dataset / DataLoader / nn.Module / training loop / validation loop / checkpointing) so that when Lightning removes it in Week 4 I understand what it's abstracting over.

## Deliverables
- `week01_pytorch_basics.ipynb` — the rewritten assignment, running end-to-end.
- `REFLECTION.md` — three short sections: what I built, what I learned, where I got stuck.

## Hard constraints
- Bare PyTorch only. No Lightning. No wandb. No HuggingFace.
- Must include: custom `Dataset`, `DataLoader`, `nn.Module`, optimizer step, validation loop, model checkpoint save/load.

## Not in scope
- Spectra. Metabolomics data enters in Week 2.
- Any GPU requirement. Laptop CPU is fine.
