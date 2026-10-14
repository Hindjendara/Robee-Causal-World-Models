# RoBee: From Stories to Plans

Neuro-symbolic pipeline that turns personal accounts of how people reached major life outcomes (for example, PhD admission or founding a startup) into structured **recipe graphs** and **causal graphs**, plus the draft paper describing the larger system.

Research conducted at the Qatar Computing Research Institute (QCRI), Hamad Bin Khalifa University, under the supervision of Dr. Safa Messaoud.

> **Status:** active research. The paper is a draft under preparation for ICLR 2026. This repository is private; please do not share code, data, or the draft outside the team.

---

## Overview

The pipeline takes a list of source links (blog posts, interviews, personal write-ups) and processes each one through these stages:

1. **Classification**: decide whether a link describes a trajectory toward a goal and which domain it belongs to.
2. **Extraction**: an LLM (Azure OpenAI, GPT-5.1) reads each account and produces
   * a **trajectory** (what happened, in order),
   * a **recipe graph** (the steps, branches, and fallbacks a person followed),
   * a **causal graph** (sub-goal nodes arranged in levels toward a single goal node, with edges retyped from control flow to enablement).
3. **Evaluation**: a DeepSeek judge scores each extracted graph for structure, grounding, and logical correctness.
4. **Analysis**: descriptive statistics over the evaluated outputs.
5. **Visualization**: an interactive HTML viewer for recipe and causal graphs.

Each causal node is a subset of recipe nodes. In the visualizer, a causal super-vertex shows `⊕ N steps` and expands to reveal the recipe steps it covers. Edges carry a confidence score and whether the support is explicit or implicit in the source text.

The paper (`paper/`) extends this toward a graph-conditioned dialogue agent that grows the causal world model through conversation, merges per-account graphs into a global DAG, and uses causal reinforcement learning for counterfactual career pathfinding.

---

## Repository structure

```
robee/
├── README.md
├── .gitignore
├── .env.example              # template for API keys (never commit .env)
├── paper/                    # ICLR 2026 draft (LaTeX)
│   ├── main.tex
│   ├── introduction.tex
│   ├── related_work.tex
│   ├── approach.tex
│   ├── math_commands.tex
│   ├── iclr2026_conference.sty / .bst / .bib
│   └── figures/
├── notebooks/
│   ├── causality.ipynb           # extraction: links → recipe_output → causal_output
│   ├── eval.ipynb                # LLM judge evaluation
│   └── analysis_and_eval.ipynb   # descriptive analysis of evaluation results
├── prompts/
│   ├── prompt_causal_extraction_v1.txt
│   ├── prompt_causal_extraction_v6.txt
│   └── prompt_causal_extraction_v7.txt   # latest
├── visualizer/
│   ├── current_visualizer.html       # latest viewer
│   └── graph_visualizer_final.html   # earlier version
├── data/
│   ├── merged_data/          # one CSV per sheet tab
│   └── evaluation/           # one CSV per sheet tab
└── docs/
    ├── evaluation_methodology.md
    └── ideas.md
```

---

## Getting started

### 1. Clone

```bash
git clone https://github.com/<your-username>/robee.git
cd robee
```

### 2. API keys

Copy the template and fill in your own keys:

```bash
cp .env.example .env
```

The notebooks read keys from environment variables (or from Colab Secrets when run in Colab). Keys are never stored in the notebooks themselves.

### 3. Run the notebooks

The notebooks were written for **Google Colab**. To run one:

1. Open [colab.research.google.com](https://colab.research.google.com) → **File → Open notebook → GitHub**, and pick the notebook from this repo.
2. Add your keys under the 🔑 **Secrets** panel in the left sidebar, using the names in `.env.example`.
3. Run cells top to bottom.

The extraction notebook expects a Google Sheet with a `links` tab and writes results row by row into `recipe_output` and `causal_output` tabs. A snapshot of that sheet is in `data/merged_data/`.

### 4. Open the visualizer

Open `visualizer/current_visualizer.html` in any browser and load a graph JSON exported by the extraction notebook.

### 5. Build the paper

Open `paper/main.tex` in Overleaf (upload the `paper/` folder as a zip) or compile locally:

```bash
cd paper
latexmk -pdf main.tex
```

---

## Prompts

Extraction prompts are versioned in `prompts/`. `v7` is the current version used for the latest runs. Older versions are kept for reproducibility of earlier evaluation results.

## Data

`data/` contains snapshots of the Google Sheets used during the project, exported as CSV (one file per tab). The source accounts are public web pages; the sheets store links, extracted graphs, and judge scores.

## Evaluation

See `docs/evaluation_methodology.md` for the scoring rubric and judge setup.

---

## Team

* Hind (CMU-Q, QCRI research intern)
* Mugur (QCRI)
* Achref
* Supervisor: Dr. Safa Messaoud (QCRI)
