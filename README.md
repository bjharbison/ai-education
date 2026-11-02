# AI Education

A self-directed curriculum for AI security practitioners — specifically, people who come from the threat-intelligence tradition and need to cross the ML vocabulary floor without converting to ML engineering.


---

## Structure

```
curriculum/         Six-part interactive curriculum (HTML)
ml-fundamentals/    Practitioner-focused ML reference + Jupyter notebooks
research/           Analysis artifacts: HBM supply chain, adversarial AI researchers
assets/             EPUB compilation of the full curriculum
```

---

## Curriculum (`curriculum/`)

Six standalone HTML files, each self-contained and readable in a browser with no server required.

| File | Content |
|---|---|
| `Part1_Understanding_AI.html` | AI foundations — what these systems are, how they work |
| `Part2_Mastering_AI_Curriculum.html` | Six-module course with 29 lessons |
| `Part3_HBM_Roadmap.html` | High Bandwidth Memory learning roadmap — 13 lessons, 4 phases |
| `Part4_Hands_On_Labs.html` | Eight hands-on labs |
| `Part5_Semiconductor_Strategy.html` | Semiconductor companion — silicon to intelligence |
| `Glossary.html` | Reference glossary |

---

## ML Fundamentals (`ml-fundamentals/`)

Written for someone who is an experienced security practitioner and a true Python beginner. The goal is the vocabulary floor, not implementation depth.

### `ML_Fundamentals_for_AI_Security.md`
The master curriculum document. Tiered by interview priority:

- **Tier 1** (interview-critical, one weekend): backprop/gradient descent, loss curves, train/val/test splits, numerical stability and log space, perplexity/cross-entropy, the training pipeline as a threat model, three ways to change a model
- **Tier 2** (background pace): tokenization/embeddings, transformers conceptually, sampling parameters, RAG, agentic loops, evaluation methodology, quantization/distillation
- **Tier 3** (only on a real signal from a real req): PyTorch hands-on, RL depth, formal adversarial ML

Every Tier 1 concept is written as *what it is → why a red teamer cares*. The second half is what generic tutorials omit.

### `Reading_Numerical_Python.md`
A 40-minute guide to reading numpy code without writing it. Covers the one idea (vectorization), reading chained expressions inside-out, and the vocabulary that appears in notebook 01. Not a Python tutorial — aimed at the specific failure mode where an ML person puts code on a shared screen and you lose the thread on line 3 for reasons unrelated to the concept.

### `notebooks/`

| Notebook | Covers | Time |
|---|---|---|
| `01_numerical_stability_perplexity.ipynb` | Float underflow, log-space arithmetic, perplexity, membership inference | 30 min |
| *(02 — planned)* | ATLAS YAML → SQLite, coverage-by-tactic queries | 60 min |
| *(03 — planned)* | Loss curve diagnostics | 45 min |

Run notebooks in JupyterLab. Requires Python 3.12+, numpy, matplotlib.

---

## Research (`research/`)

Standalone analysis artifacts.

### `HBM_Supply_Chain.html`
Interactive visualization of the global High Bandwidth Memory supply chain: 6 layers from raw materials through end customers, with a parallel China domestic track (CXMT, Huawei/XMC/SJSemi, SMIC). Clickable nodes with detail panels, risk scores, market share data, and chokepoint analysis. CoWoS packaging is the primary constraint.

### `Minds_That_Made_AI_Adversarial.html`
Profiles of researchers working at the intersection of ML and adversarial/security research: Carlini, Tramèr, Zou, Song, Greshake, Ippolito, Nasr, Jagielski, Rando, Qi, Labonne. Organized by primary contribution area — extraction, evasion, poisoning, prompt injection, model editing.

---

## Assets (`assets/`)

`AI_Systems_Complete_Reference.epub` — EPUB3 compilation of the full curriculum, readable on any e-reader or with Apple Books / Kindle.

---

## What this is not

This is not a course for ML engineers. It does not cover PyTorch, CUDA, training loops, or model architecture in implementation depth. It is built for practitioners in the threat-intelligence tradition who get interviewed by the ML-research tradition — and need the shared vocabulary floor without changing which column they're in.
