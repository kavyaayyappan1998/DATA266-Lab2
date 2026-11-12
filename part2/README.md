# DATA 266 Lab 2 - Part 2: Fine-tuning Stable Diffusion with LoRA (Team ##)

Fine-tunes Stable Diffusion 1.5 on a captioned character dataset, compares two adapter runs with the base model,
and evaluates quality, prompt alignment, diversity and cost.

## Layout
```
notebooks/p2_finetune.ipynb   single self-contained notebook (Kaggle T4 or Colab T4)
prompts/p2_prompts.json       5 out-of-domain + 5 compositional prompts, fixed before training
prompts/p2_prompts_final.json written by the notebook: the full 20 prompts (commit before training ends)
outputs/                      everything the notebook produces (git-ignored; weights and images go to Hugging Face)
```

## Defaults (change in Step 0 of the notebook)
| Setting | Value |
|---|---|
| Dataset | `reach-vb/pokemon-blip-captions` (or `lambdalabs/naruto-blip-captions`) |
| Base model | `stable-diffusion-v1-5/stable-diffusion-v1-5`, 512x512 |
| Run 1 | LoRA rank 4 |
| Run 2 | LoRA rank 16 (set `"dora": True` for DoRA) |
| Budget | 1500 steps, batch 2 x 2 accumulation, lr 1e-4 cosine, fp16, seed 266 |
| Generation | DPMSolverMultistep, 30 steps, guidance 7.5, seeds 0 and 1 |
| Monitoring | held-out denoising loss every 250 steps (fixed noise), adapter and training-state checkpoint every 500 steps (an interrupted run resumes from it) |
| Extra KID set | 60 additional test captions per model, reported beside the required 20-image KID |

## Run
1. Set `TEAM` and `HF_USER` in Step 0; add a Hugging Face **write** token as a secret named `HF_TOKEN`.
2. First time: set `QUICK_TEST = True` and Run All (a few minutes) to check everything executes. Then set it back to `False`
   and Run All for the real run (the notebook detects the changed settings and retrains and regenerates by itself).
3. After the run: have 3 raters fill `human_eval/final_vs_base/ratings.csv`, rerun Step 12, then write Step 15.

## Outputs
`cleaning_report.json`, `duplicates_removed.csv`, `dataset_meta.csv`, `run_table.csv`, `results_table.csv`, `generation_scores.csv`, `grid_base_vs_runs.png`,
`generated/<model>/*.png` + `metadata.jsonl`, `human_eval/pair_XX.png` + key and rating template, failure-mode figures,
`memorization_check.csv`, `memorization_pairs.png`, `requirements_frozen.txt`.

## Before tagging `p2-v1.0`
- [ ] Commit `prompts/p2_prompts_final.json` before training finishes
- [ ] Both models on Hugging Face, model card on the final run, generated images dataset uploaded
- [ ] Human evaluation with at least 3 raters (one outside the team)
- [ ] Notebook saved with all outputs; dataset description and conclusion written in your own words
