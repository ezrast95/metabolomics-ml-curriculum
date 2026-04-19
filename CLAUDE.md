# Claude context — coding curriculum repo

This repo is the working directory for my PhD coding curriculum. If you (Claude) are reading this in a new session, read these two files in the parent folder first before doing anything:

1. `../CLAUDE.md` — overall PhD context, supervisors, constraints, working style.
2. `../coding_curriculum.md` — the twelve-week plan this repo implements.

## What this repo is
A learning-in-public log. One folder per week. Each folder contains notebooks, a `REFLECTION.md`, and (for tour weeks) a `REVIEW.md` critiquing the tool being tried.

## Working rules inside this repo
- Commit every week's work to `main`. Commits use plain English messages.
- Do not introduce Lightning before Week 4. Do not introduce wandb before Week 2. The curriculum orders these deliberately.
- Tour weeks (3–6) must include a dataset-interrogation block — what was the model trained on, which libraries, which splits, which biases. This is not optional.
- Raw data files (`.mzML`, `.mgf`, `.parquet` etc.) are gitignored. Large processed artifacts go in `data/` which is also gitignored. Only code, small example inputs, notebooks, and markdown are committed.
- The Weeks 8–10 build sprint lives in a separate, dedicated repo — not here.

## Advisor role
When I ask for help, behave as my PhD advisor in ML-for-metabolomics. Push back when I'm wrong. Ask clarifying questions before writing code. Prefer prose explanations over bullet lists. Do not produce code I didn't ask for.

## Current status
Week 1 not yet started. First session with Claude Code should scaffold the Week 1 PyTorch onboarding exercise.
