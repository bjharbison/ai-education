# Course infographics

Rendered PNGs live here; their sources live in `src/`.

| Image | Teaches | Linked from |
|---|---|---|
| `01-parallelism.png` | Data, tensor, pipeline and ZeRO/FSDP parallelism: what each GPU holds, sees and syncs | `curriculum/Part1_Understanding_AI.html`, Distributed Training |
| `02-rlhf-vs-dpo.png` | RLHF with PPO vs DPO, and why both keep a KL anchor | `curriculum/Part2_Mastering_AI_Curriculum.html`, RLHF & Alignment (after the DPO code) |
| `03-backprop-vs-evasion.png` | Backprop with respect to the weights (training) vs the input (evasion) | `ml-fundamentals/ML_Fundamentals_for_AI_Security.md`, §1 |
| `04-data-splits.png` | Train / validation / test jobs, and data leakage | `ml-fundamentals/ML_Fundamentals_for_AI_Security.md`, §3 |
| `05-pipeline-attacks.png` | The LLM pipeline stages and where each attack lands | `ml-fundamentals/ML_Fundamentals_for_AI_Security.md`, §6 |

HTML pages reference images with a relative path (`../docs/img/…`), so they show when the repo is opened locally. The EPUB in `assets/` is unchanged.

To regenerate one:

```bash
pip install playwright && playwright install chromium
python3 docs/img/src/render.py docs/img/src/04-data-splits.json docs/img/04-data-splits.png
```

Fonts: Inter and DejaVu Sans Mono (`apt install fonts-inter fonts-dejavu` if missing).

Layouts: `steps` (an order things happen in, with an optional written-order strip), `cards` (a 2-column grid of building blocks or comparisons), `compare` (two side-by-side panels, wrong vs right), and `matrix` (rows × columns). `render.py`'s docstring lists the spec keys. The renderer is shared across Brian's learning repos; keep one visual language.

Each `src/*.json` spec holds the text. Every fact must come from the page it's linked from. If that page changes, update the spec and re-render.
