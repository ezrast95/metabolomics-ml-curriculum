# Claude context — coding curriculum repo

This repo is the working directory for my PhD coding curriculum (ML for metabolomics, Brotman lab, TAU). The files below are imported automatically every session — no need to ask me to re-explain background, working style, or the curriculum plan.

@claude/knowledge_state.md
@claude/curriculum.md
@claude/coding-partner.md

## What this repo is
A learning-in-public log. One folder per section. Each folder contains a README.md file explaining the assignment of the section, jupyter notebooks implementing the assignmnet.

## Working rules inside this repo
- Commit every section's work to `main`. Commits use plain English messages.
- Do not introduce Lightning before section 4. Do not introduce wandb before section 2. The curriculum orders these deliberately.
- Tour sections (3–6) must include a dataset-interrogation block — what was the model trained on, which libraries, which splits, which biases. This is not optional.
- Raw data files (`.mzML`, `.mgf`, `.parquet` etc.) are gitignored. Large processed artifacts go in `data/` which is also gitignored. Only code, small example inputs, notebooks, and markdown are committed.

## Current status
See the progress table in `claude/curriculum.md`. section 1 not yet started as of writing — first session with Claude Code should scaffold the section 1 PyTorch onboarding exercise.
