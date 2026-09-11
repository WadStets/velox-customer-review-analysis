# Velox Foods: AI-Powered Customer Review Analysis

Aspect-Based Sentiment Analysis (ABSA) pipeline comparing a **local LLM (Ollama)** against a
**cloud LLM (Groq)** for extracting structured customer feedback from restaurant reviews —
built for a fictional fast-food chain, Velox Foods, evaluating whether an AI-driven review
analysis pilot is worth scaling to production.

## Business Context

Velox Foods has 514 franchised restaurants. Star ratings alone don't explain *why* some
locations are underperforming — the "why" lives in the free-text review body. This project
builds an automated pipeline to extract structured sentiment on **Food, Service, Price,
Ambiance**, and **Overall Rating** from raw customer reviews, then compares a free local model
against a paid cloud model on accuracy, reliability, speed, and cost — to give an
Operations Director a clear, numbers-backed recommendation.

📄 **[Read the full report](report/Velox_Foods_Report_Customer_Review_Analysis.pdf)**

## Key Findings

| Metric | Ollama (Local) | Groq (Cloud) |
|---|---|---|
| Avg. MAE (lower = better) | 0.494 | **0.296** (~40% lower error) |
| Avg. Recall (higher = better) | 75% | **84%** |
| Time to process 50,000 reviews | 91.1 hours (~3.8 days) | **5.6 hours** |
| Estimated cost (50,000 reviews) | $0 (compute only) | **~$35.11** |

**Recommendation:** Groq Cloud API — the accuracy and speed gains far outweigh the modest
per-token cost, and the local model's slower, less consistent processing ties up a laptop for
days at a time.

## Repository Structure

```
├── notebooks/
│   ├── velox_ollama_pipeline.ipynb      # Local ABSA pipeline (Ollama)
│   ├── velox_groq_pipeline.ipynb        # Cloud ABSA pipeline (Groq)
│   ├── velox_accuracy_metrics.ipynb     # MAE & Recall vs. human-labelled ground truth
│   └── velox_operational_metrics.ipynb  # Speed, reliability & cost analysis
├── data/
│   ├── reviews.csv                      # Raw customer reviews (input)
│   ├── labeled_reviews.xlsx             # Human-labelled ground truth (50 reviews)
│   ├── ollama_aspects.csv               # Ollama sentiment predictions
│   ├── ollama_metadata.csv              # Ollama timing & token metadata
│   ├── ollama_summary.json
│   ├── groq_aspects.csv                 # Groq sentiment predictions
│   ├── groq_metadata.csv                # Groq timing & token metadata
│   ├── groq_summary.json
│   ├── accuracy_comparison.png
│   └── speed_boxplot.png
├── report/
│   └── Velox_Foods_Report_Customer_Review_Analysis.pdf
└── environment.yml                      # Conda environment for reproducing the pipeline
```

## Methodology

1. **Local pipeline** — 50 customer reviews processed with a small open-source model via
   [Ollama](https://ollama.com), run entirely on a local machine (free, private, slower).
2. **Cloud pipeline** — the same 50 reviews processed with `llama-3.3-70b-versatile` via the
   [Groq API](https://groq.com) (usage-based cost, much faster, higher-capacity model).
3. Both pipelines use an identical system prompt and a **Pydantic**-enforced JSON schema:
   `overall_rating` (required) plus four optional aspects — `food`, `service`, `price`,
   `ambiance` — each scored from -1.0 to +1.0, or `null` if not mentioned in the review.
4. A random sample of 50 reviews was **hand-labelled** by a human analyst to serve as ground
   truth for scoring both models.
5. Accuracy was measured with **Mean Absolute Error (MAE)** against the human labels, and
   **Recall** (did the model correctly detect that an aspect was mentioned at all?).
6. Operational viability was measured via processing time, token usage, and extrapolated cost
   and throughput at Velox Foods' real-world scale (514 restaurants).

## Setup

```bash
conda env create -f environment.yml
conda activate velox-sentiment-env
```

To run the cloud pipeline, you'll need a free [Groq API key](https://console.groq.com/keys)
saved in a `.env` file (not committed to this repo) as `GROQ_API_KEY=your_key_here`.

## Tech Stack

Python · pandas · Pydantic · Ollama · Groq API · matplotlib · seaborn · Jupyter
