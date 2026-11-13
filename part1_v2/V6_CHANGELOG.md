# Part 1 v6 experimental changes

- Preserves the original `p1_rag_v5.ipynb` unchanged.
- Adds `p1_rag_v6.ipynb` with expanded figure-image evidence for comparison questions (up to 3 figures).
- Ensures explicitly named figures are included in evidence without changing the ranked retrieval list used for Recall@5.
- Adds a generic numerical-reasoning checklist to the answer prompt for two-source comparisons, user-supplied assumptions, and corrections.
- No golden-set answers, question IDs, or expected values are hardcoded.
- Static Python syntax check passed. End-to-end inference and accuracy are untested.

## Validation
Run the notebook on RunPod, compare `outputs/results_table.csv` and golden outputs with v5, and inspect any regressions. This experimental change does not guarantee 23/25. Keep v5 as baseline.
