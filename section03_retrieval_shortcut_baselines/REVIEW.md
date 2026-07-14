# Section 3 — review notes

Running notes on prior work, read closely enough to avoid duplicating it or repeating its mistakes,
not necessarily reproduced in full. See `claude/curriculum.md` update log (2026-07-14 entry) for how
this section's scope was decided.

## Code availability check: Appendix A.4 baselines (MassSpecGym v1.5 pitfalls paper) (2026-07-14)

Before implementing "PubChem Default Ranking" (Method 5) and "Ranking by Num. of Chiral Atoms"
(Method 6) from `papers/MassSpecGym v1.5 common pitfalls.pdf` (Appendix A.4), checked whether the
paper's own reference implementation is public, rather than relying on the paper's prose description
alone.

**Searched** (full clone of `github.com/pluskal-lab/MassSpecGym`, the commit pinned in Section 2):
- Every `.py`/`.ipynb`/`.md`/`.sh` file in the repo for `pubchem`, `chiral`, `CID`, `deposition`.
- The full retrieval model registry (`massspecgym/models/retrieval/__init__.py`) and `run.py`'s
  model dispatcher — the complete list of implemented retrieval baselines is `RandomRetrieval`,
  `RandomRetrievalGTFormula`, `DeepSetsRetrieval`, `FingerprintFFNRetrieval`, `FromDictRetrieval`.
- `notebooks/massspecgym_in_the_wild/massspecgym_v1.5_validation.ipynb` — named to match this exact
  paper, but its actual content only validates that the `DeepSets+Fourier` baseline trains identically
  on v1 vs. v1.5 data (a data-migration sanity check), not the adversarial stress-test baselines.
- The paper itself for a code/data-availability statement pointing to a separate reproduction
  archive — none found. The only Zenodo DOI in the references is an unrelated RDKit software citation.

**Result:** neither baseline's scoring code is publicly available anywhere I could find. The only
`chiral`-related code in the repo is a generic `AtomChiral` one-hot GNN feature encoder — unrelated
infrastructure, not this ranking method.

**Implication:** Appendix A.4's prose is the most precise public specification that exists for these
two baselines:
- **Chiral atoms (Method 6)**: fully unambiguous — count atoms per candidate with
  `GetChiralTag() != CHI_UNSPECIFIED` via RDKit, rank by that count, direction (ascending/descending)
  decided empirically from whichever direction the training-set ground-truth/candidate split favors.
- **PubChem default ranking (Method 5)**: the phrase "CID deposition count" is not further specified.
  Best working interpretation: number of Substance IDs (SIDs) mapping to a CID via PUG-REST
  (`compound/cid/<cid>/sids/JSON`, count of returned list) as a "how well-known is this compound"
  proxy — but this is an inference, not a verified fact, since there's no code to check it against.
  Sanity-check against 2-3 known examples (e.g. glucose should score high) before trusting it at the
  full test-fold scale (782,695 unique candidate molecules in the v1.5 test fold).

Worth a line in whatever comes out of this section's own review: the exact scoring semantics of two
baselines central to this paper's headline claim aren't independently reproducible from anything
publicly released — a little ironic given the paper's whole subject is implementation divergence and
reproducibility (§6.2 of the same paper).
