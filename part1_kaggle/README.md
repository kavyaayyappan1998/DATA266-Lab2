# DATA 266 Lab 2 - Part 1: Multimodal RAG (Team 48)

Answers questions about three economics PDFs (text, figures, tables) with a read-only knowledge base,
per-conversation memory, and guardrails against the untrusted Appendix C.
Kaggle competition: DATA 266 Fall 2026 - Lab 2 Part 1: Multimodal RAG.

## Layout
```
notebooks/p1_rag.ipynb   single self-contained notebook (runs top to bottom on Kaggle and writes submission.csv)
eval/golden_set.json     25 golden questions written by the team (not the competition questions)
eval/human_eval_TEMPLATE.csv
data/                    lab files for local runs (git-ignored: Kaggle provides them)
outputs/                 created by the run: figure crops, cached chart readings, result tables
requirements.txt
```
This folder contains no results. Everything under `outputs/` and `submission.csv` are produced by running the notebook.

## Pipeline
1. Load manifest, PDFs, questions  2. Clean text  3. Chunk pages; crop every Figure/Table by caption; extraction inventory check
4. Visualize the knowledge base  5. Load bge-small, bge-reranker-base, Qwen2.5-VL-3B
6. The vision model describes each figure and reads the values printed on it; a reading that lost its category labels is repaired from the description
7. Index in ChromaDB (doc_id, page, figure_id, trust_level) + BM25
7b. Structured fact store: every table cell, plus labelled bar charts whose series the tables do not report; rule-based resolver for forecast vs outturn, stated revisions and conflicts
8. Retrieval: dense baseline, hybrid (weighted RRF) + exact figure lookup, optional cross-encoder reranker
9. Rules: low-trust filter, sentence-level injection removal, evidence headers with trust / vintage / revision keywords, output validation of citations and figure references
10. Conversation memory: structured session values; permanent edits refused; "that year" follows the year last named
11. Golden-set evaluation of four systems  12. submission.csv (validated)  13. Conclusion

## Rules applied after the model answers (none looks at a specific question)
| Rule | What it does |
|---|---|
| conflict / revised / outturn_vs_forecast | Status and both values come from the fact store; the forecast error is computed when the question asks for it |
| false_premise | A "why did X rise / fall / contract" question is checked against the facts first |
| year_change | The change between two years of one series is computed in code |
| indicator_gap | The difference between two indicators of the same entity and year is computed in code |
| figure_vintage | A figure of an older data vintage shows the forecast for years its vintage had not seen |
| figure_correction | A corrected figure supersedes the original; the changed year is taken from the question or the erratum text |
| figure_comparison | Two labelled bar charts named in the question are compared category by category |
| session_override / permanent_refusal | Arithmetic with the user's value is done in code; the knowledge base is never changed |
| named labels | A figure or table the user names is always referenced and cited; other figures the model cited are dropped |

## Run on Kaggle (the graded path)
1. Join the competition, then create a notebook from the competition page so its data is attached under `/kaggle/input`.
2. File > Import Notebook > upload `notebooks/p1_rag.ipynb`.
3. Settings: Accelerator **GPU T4**, Internet **On** (the models are downloaded from Hugging Face).
4. Optional: attach a dataset containing `eval/golden_set.json` to get the Step 11 tables; without it Step 11 is skipped and the submission is unchanged.
5. Save Version > **Save & Run All (Commit)**. When it finishes, open the version > Output > `submission.csv` > Submit to Competition.
6. Select that submission as final on the leaderboard. The notebook file must be the one at Git tag `p1-v1.0`.

The notebook reads `QUESTIONS_PATH` (default `Questions.csv` from the competition data) and writes `/kaggle/working/submission.csv`
in the required format: `id,conversation_id,question_id,answer,figure_references,citations,status`, lists joined by commas without spaces, `NONE` for an empty list.
Expected runtime on a T4 is well under the 90-minute limit; the last line of Step 12 prints `submission.csv is valid: 40 rows`.

**Local / rented GPU:** copy the lab files into `data/`, then from `notebooks/`:
```
jupyter nbconvert --to notebook --execute --inplace p1_rag.ipynb --ExecutePreprocessor.timeout=-1
```

## Models (all local, no hosted API)
- Embeddings: `BAAI/bge-small-en-v1.5`
- Reranker (optional, bonus): `BAAI/bge-reranker-base`
- Figure reading and answers: `Qwen/Qwen2.5-VL-3B-Instruct` (fp16, frozen, greedy decoding, seed 266)

## Before tagging `p1-v1.0`
- [ ] Verify the 25 golden questions against the PDFs (two answers are marked as unverified guesses in earlier notes: G15.3 and G16.2)
- [ ] Fill in `CONTRIBUTIONS.md` and review `AI_USAGE.md`
- [ ] Run All on Kaggle with outputs saved; download the executed notebook and commit it as `notebooks/p1_rag.ipynb`
- [ ] Set `FINAL_MODE` in Step 0 from the Step 11 table if another system is better, and rerun
- [ ] Both members fill `human_eval.csv` (copy of `outputs/human_eval_template.csv` with `rater_a`, `rater_b`), save it in `eval/`
- [ ] Fill in Step 13 (conclusion) from the notebook outputs

## Evaluation method (Step 11)
- Correct = status matches, every gold number appears within +/-1% with the right sign, required phrases present, forbidden content absent.
- Recall@5 = a gold source (`DOCx:pN` or `DOCx:Figure/Table label`) is among the top 5 retrieved items.
- Citation accuracy = cited sources that are gold sources (a gold figure also covers its own page), pooled over all answers.
- Efficiency = median seconds per question. Human evaluation = two raters, 1-5 groundedness, agreement within one point.

## Known limits
- A 3B vision model estimates values on line charts that have no printed labels, so only labelled bar charts are turned into facts; other chart questions are answered from the image.
- Comparisons that need a value stated only in running text (not in a table) still depend on the language model.
- Injection defence is trust filtering plus pattern-based sentence removal, not a learned classifier.
