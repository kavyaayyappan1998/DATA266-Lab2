# DATA 266 Lab 2 - Part 1: Multimodal RAG (Team ##)

Answers questions about three economics PDFs (text, figures, tables) with a read-only knowledge base,
per-conversation memory, and guardrails against the untrusted Appendix C.

## Layout
```
notebooks/p1_rag.ipynb   single self-contained notebook (runs top to bottom on Kaggle)
eval/golden_set.json     25 golden questions (DRAFT - verify every answer against the PDFs, then commit before tuning)
data/                    lab files for local runs (git-ignored)
outputs/                 figure crops, cached figure descriptions, result CSVs
requirements.txt         pinned versions
```

## Pipeline
1. Load manifest, PDFs, questions  2. Clean text  3. Chunk pages; crop every Figure/Table by caption; extraction inventory check
4. Visualize the knowledge base  5. Load bge-small, bge-reranker-base, Qwen2.5-VL-3B  6. VLM writes a description per figure
7. Index in ChromaDB (doc_id, page, figure_id, trust_level) + BM25
8. Retrieval: dense baseline, hybrid (RRF) + exact figure lookup, optional cross-encoder reranker
9. Rules: low-trust filter, sentence-level injection removal, evidence headers with trust/vintage/revision keywords,
   manifest precedence policy in the prompt, output validation of citations and figure references
9b. Structured fact store (table cells read from the PDF text layer) and a rule-based resolver for forecast vs outturn,
   stated revisions and conflicts; fact-level citation check (cited source must hold the value for the same year and entity)
10. Conversation memory: structured session values (variable, value, unit, scope, turn); permanent edits refused
11. Golden-set evaluation of three systems (per-question checks, pooled citation accuracy)  12. submission.csv  13. Conclusion

Outputs written by the notebook: `extraction_inventory.csv`, `extraction_thumbnails.png`, `fact_store.csv`, `figure_descriptions.json`, `results_table.csv`,
`golden_results_<system>.csv`, `human_eval_template.csv`, `requirements_frozen.txt`, `submission.csv`.

## Run
**Kaggle (graded path):** new notebook > upload `notebooks/p1_rag.ipynb` > attach the competition data
(and a dataset containing `golden_set.json`) > GPU T4 > Run All. It reads `QUESTIONS_PATH`
(default `Questions.csv`) and writes `submission.csv` with no manual step.

**Local:** copy the lab files into `data/`, `pip install -r requirements.txt`, run the notebook from `notebooks/`.

## Models (all local, no hosted API)
- Embeddings: `BAAI/bge-small-en-v1.5`
- Reranker (optional, bonus): `BAAI/bge-reranker-base`
- Figure descriptions and answers: `Qwen/Qwen2.5-VL-3B-Instruct` (fp16, frozen, greedy decoding)

## Before tagging `p1-v1.0`
- [ ] Verify the 25 draft golden questions against the PDFs, reword them in your own words, and commit before tuning
- [ ] Fill in `CONTRIBUTIONS.md` and review `AI_USAGE.md`
- [ ] Set `FINAL_MODE` in Step 0 from the Step 11 table; copy `requirements_frozen.txt` into `requirements.txt`
- [ ] Run All on Kaggle with outputs saved; check `validate_submission.py` passes
- [ ] Both members fill `human_eval.csv`
- [ ] Fill in Step 13 (conclusion) from the notebook outputs

## Evaluation method
- Correct = status matches, every gold number appears within +/-1%, required phrases present, forbidden content absent.
- Recall@5 = a gold source (`DOCx:pN` or `DOCx:Figure/Table label`) is among the top 5 retrieved items.
- Citation accuracy = cited sources that are gold sources (a gold figure also covers its own page), pooled over all answers.
- Efficiency = median seconds per question. Human evaluation = two raters, 1-5 groundedness, agreement within one point.

## Known limits
- Figures are searched through VLM-written descriptions; a 3B model can misread dense charts.
- The fact store covers the 9 tables and stated revisions in text. Values that exist only in running text or inside charts are not in it, so conflicts involving those still depend on the LLM.
- Injection defence is trust filtering plus pattern-based sentence removal, not a learned classifier.
