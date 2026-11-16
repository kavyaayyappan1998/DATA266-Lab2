# DATA 266 Lab 2 - Part 3: Multimodal Multi-Agent System (Team 48)

Five agents that understand an input image, turn a request into a prompt, generate a new image, check it, and revise it if needed.

```
request -> Planner (plan + input safety) -> Vision Understanding -> Prompt Engineering -> Image Generation -> Critique (output safety)
                                                  |                        ^                                   |
                                   caption/question stops here             +--------- revise (max 2) ----------+
```

## Layout
```
notebooks/p3_agents_dinosaurs_only.ipynb   single self-contained notebook (agents, orchestration, evaluation, Gradio app)
eval/p3_scenarios.json      15 scenarios with rubrics, on the dinosaur test images
eval/p3_redteam.json        10 red-team cases
outputs/                    created by the run: test images, generated images, JSON traces, result tables, figures (git-ignored)
```
This folder contains no results. Everything under `outputs/` is produced by running the notebook.

## Models (all local, no hosted API)
| Role | Model |
|---|---|
| Vision, planner pre-scan, critic verdict | Qwen2.5-VL-3B-Instruct |
| Style change | SDXL base 1.0 at 1024 px when the GPU has 20 GB or more; Stable Diffusion 1.5 otherwise (the 16 GB path) |
| Domain (Pokemon) style | Stable Diffusion 1.5 + the Part 2 LoRA `kavyaayyappan1990/data266-lab2-team48-run2-lora-r16` |
| Edits | InstructPix2Pix, applied inside a mask when the target object is located |
| Critic scores | CLIP ViT-B/32 |
| Safety | Stable Diffusion safety checker with a CLIP second opinion |

Enhancement uses deterministic image operations.

## Test images
Every test picture is a dinosaur image. Five well-known species are rendered as photorealistic scenes by the system's own generator (Step 6b).
If the Hugging Face dataset `bigdata-pw/Dinosaurs` can be reached, its reference drawings fix the poses; otherwise the species are rendered from their names alone.
The images are **generated, not photographs**; the notebook's table says so and the report must too.
From them the notebook builds two single-dinosaur scenes, a 2 x 2 picture of four dinosaurs (counting), a scene with a park sign (text reading), a dulled copy (enhancement) and three copies with instructions written into them (red-team).

## What the notebook runs
| Step | Content |
|---|---|
| 6b | Render the dinosaur test images, show them, self-test every model on them |
| 10 | The 15 scenarios, with and without the critique loop; mask on/off; SD 1.5 vs SDXL |
| 10b | Every rendered species, eight requests each (describe, name, four styles, sky edit, enhance) |
| 10c | Evidence figures: what the critic did, what the mask did, what the generator changed |
| 11 | Red-team: 10 cases |
| 12 | Human evaluation and critic agreement (reads the two files you fill in) |
| 13 | Gradio demo app |

## Settings (Step 0 of the notebook)
| Setting | Meaning |
|---|---|
| `LORA_REPO` | Hugging Face id of the Part 2 final model |
| `USE_SDXL` | `auto` (default), `1` or `0` |
| `LOCAL_EDIT` | Mask-based editing on (`1`) or off (`0`) |
| `N_PHOTOS` | Number of dinosaur species rendered (at least 4 are needed for the counting picture to show four different animals) |
| `RUN_EVAL`, `RUN_SHOWCASE` | Run the evaluation steps / the wider demonstration in Step 10b |
| `LAUNCH_APP` | Start the Gradio app at the end. Keep off for a headless Run All |

## Run
Shut down every other notebook kernel first (a second kernel on the GPU makes SDXL run out of memory), then from `notebooks/`:
```
jupyter nbconvert --to notebook --execute --inplace p3_agents_dinosaurs_only.ipynb --ExecutePreprocessor.timeout=-1
```
About 40 minutes on a 24 GB GPU, including the SDXL download (about 7 GB of disk).

**Live demo:** set the environment variables `LAUNCH_APP=1` and `RUN_EVAL=0`, Run All, open the public link Gradio prints.

## After the run
1. Each member fills `outputs/human_eval_TEMPLATE.csv` alone; save as `eval/p3_human_eval.csv`.
2. Both review `outputs/redteam_review_TEMPLATE.csv`; save as `eval/p3_redteam_review.csv`.
3. Fill `member_A_pass` / `member_B_pass` in `outputs/photo_results.csv`.
4. Rerun Step 12 for task success rate, member agreement, critic agreement and the reviewed attack success rate.
5. Write Step 14 (conclusion and system card) from the outputs.

## Known limits
Routing and safety are English keyword rules and can be evaded by rephrasing; a 3B vision model can misread small text;
masks need the vision model to locate the target, otherwise the edit covers the whole image; SDXL does not fit the 16 GB limit together with the vision model, so it is optional.

## History
The scenario and red-team files were rewritten on 15 Nov 2026 to use dinosaur images instead of the original drawings (house, apples, shop sign). The requests, categories and pass rules kept the same structure; state this in the report.
