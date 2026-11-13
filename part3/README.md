# DATA 266 Lab 2 - Part 3: Multimodal Multi-Agent System (Team ##)

Five agents that understand an input image, turn a request into a prompt, generate a new image, check it, and revise it if needed.

```
request -> Planner (input safety) -> Vision Understanding -> Prompt Engineering -> Image Generation -> Critique (output safety)
                                             |                        ^                                   |
                              caption/question stops here             +--------- revise (max 2) ----------+
```

## Layout
```
notebooks/p3_agents.ipynb   single self-contained notebook (agents, orchestration, evaluation, Gradio app)
eval/p3_scenarios.json      15 test scenarios with rubrics (written before running)
eval/p3_redteam.json        10 red-team cases (written before running)
outputs/                    assets, generated images, JSON traces, result tables (git-ignored)
```

## Models (all local, no hosted API)
Qwen2.5-VL-3B-Instruct (vision, planner pre-scan, critic judgement) · Stable Diffusion 1.5 image-to-image (+ the Part 2 LoRA for domain-style requests) ·
InstructPix2Pix (edits) · CLIP ViT-B/32 (critic score) · Stable Diffusion safety checker (output safety). Enhancement uses deterministic image operations.

## Settings (Step 0 of the notebook)
| Setting | Meaning |
|---|---|
| `LORA_REPO` | Hugging Face id of your Part 2 final model. If empty, domain-style requests fall back to the base model and the trace says so. |
| `DOMAIN_NAME`, `DOMAIN_WORDS` | The Part 2 domain and the words that route a request to it. |
| `RUN_EVAL` | Run the 15 scenarios (twice: without and with the critique loop) and the 10 red-team cases. |
| `LAUNCH_APP` | Start the Gradio app at the end. Keep off for a headless Run All. |

## Run
**Evaluation run (headless):** from `notebooks/`
```
jupyter nbconvert --to notebook --execute --inplace p3_agents.ipynb --ExecutePreprocessor.timeout=-1
```
**Live demo:** set `LAUNCH_APP = True` and `RUN_EVAL = False` in Step 0, Run All, open the public link Gradio prints.

## After the run
1. Each member fills `outputs/human_eval_TEMPLATE.csv` alone; save as `eval/p3_human_eval.csv`.
2. Both review `outputs/redteam_review_TEMPLATE.csv`; save as `eval/p3_redteam_review.csv`.
3. Rerun Step 12 for task success rate, member agreement, critic agreement and the reviewed attack success rate.
4. Write Step 14 (conclusion and system card) from the outputs.

## Known limits
Routing and safety are English keyword rules and can be evaded by rephrasing; a 3B vision model can misread small text;
the test inputs are drawn images, so add a few real photos of your own for a stronger evaluation.
