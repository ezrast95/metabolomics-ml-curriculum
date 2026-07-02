# Knowledge State — Ezra

*Last updated: 2026-06-30. This file is the calibration source for any new session — read it before asking Ezra to re-explain his background. Update it whenever understanding meaningfully changes.*

---

## ML / Deep Learning

### Concepts (from Andrew Ng's ML + DL Specializations — completed June 2026)

**Backpropagation**
Understands that gradients flow backward through the network to update weights, and knows PyTorch's `.grad` API. Gap: hasn't internalized the chain rule through the computational graph at the math level; hasn't implemented backprop from scratch. Knows the "what" but not yet the "how it actually computes."

**Attention mechanism**
Knows the Q/K/V architecture, that it computes relationships between all token pairs, masked vs. unmasked variants, and encoder-only / decoder-only / encoder-decoder configurations. Gap: hasn't worked through the math (scaled dot-product: QKᵀ/√d_k → softmax → ×V) or implemented from scratch.

**General posture**
Concepts from Ng's courses were taught in TensorFlow. Most architectures are understood at the "what it does" level; fewer at the "why the math works" level. Has not yet implemented any architecture from scratch in PyTorch.

### Coding

**Python** — comfortable.

**PyTorch** — has seen a training loop, `Dataset`/`DataLoader`, `nn.Module`. Has never written one from scratch without examples. Needs hands-on reps before the fundamentals are durable.

**TensorFlow** — used in Ng's courses. Not the target stack for PhD work.

---

## Metabolomics Research

### Wet-lab / instrument background
MSc with Brotman lab (Tel Aviv University): untargeted plant metabolomics, LC-MS on Thermo Orbitraps (Q Exactive, Orbitrap IQX), including MSn. Mostly wheat seeds. Primary analysis pipeline: Compound Discoverer + custom Python downstream. No in-house instruments available for PhD — public data only.

### ML for metabolomics — current understanding

**Coverage bias / shortcut learning** (core current focus)
Training data for MS annotation models is drawn almost entirely from authentic chemical standards — compounds that are commercially available, easy to synthesize or purchase, and routinely measured. This means the training distribution does not represent the true molecular space. Models may exploit shortcut learning: selecting candidates likely to appear in MS libraries rather than genuinely matching spectra to structure. Key evidence: MassSpecGym showed that a retrieval model selecting candidates via PubChem lookup *without looking at the spectra at all* achieves surprisingly strong results — suggesting models are partially rewarded for knowing what's commonly measured, not for spectral understanding.

### Papers read
- MassSpecGym v1 — NeurIPS 2024 benchmark paper (defines tasks: de novo generation, retrieval, Bayesian retrieval)
- MassSpecGym v1.5 common pitfalls (2026) — open question flagged: training data distribution doesn't cover real molecular space
- Coverage bias in chemical databases papers (`phd_literature_review/metabolite_annotation/covariage bias in chemical databases using in ml models/`)

### Current research direction
Exploring the coverage bias / distribution shift problem in MS annotation as a candidate PhD topic. Not yet committed. ~2-month deadline to a written research proposal (end of August 2026).

---

## What to calibrate to in new sessions

- Don't assume PyTorch fluency — Ezra has seen it but not built with it yet.
- Don't skip to research implications before checking understanding of the underlying paper math/method.
- The coverage bias problem is the current leading thread — but ideas are still being explored, not locked in.
