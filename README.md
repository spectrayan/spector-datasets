# Spector Memory Datasets & Cognitive Benchmarks

This repository stores the synthetic corpora, relationship graphs, queries, search relevance judgments (qrels), and evaluation results used to validate the **Spector Memory** cognitive retrieval engine.

Spector Memory is a biologically-inspired agentic memory system designed to model human recall mechanics (importance, arousal-modulated decay, tag gating, and associative graph traversal) rather than simple semantic proximity.

---

## 📂 Repository Layout

The repository is structured into two main dataset tracks, separating standard neurotypical baseline behavior from neurodivergent ADHD configurations:

```text
D:\git\spector-datasets\
  ├── balanced-baseline/              # Standard Family (balanced PM persona)
  │     ├── data/                     # Root data and graph files
  │     │     ├── corpus.jsonl         # Combined daily + biographical memories (11.3k records)
  │     │     ├── corpus-biographical.jsonl
  │     │     ├── entities.jsonl
  │     │     ├── temporal_chains.jsonl
  │     │     ├── hebbian_edges.jsonl
  │     │     ├── queries.jsonl        # Evaluation queries (19 standard presets)
  │     │     ├── qrels.tsv            # Query relevance ground-truth judgments
  │     │     ├── persona.json         # Mike Thompson neurotypical profile definitions
  │     │     ├── validation-report.txt
  │     │     └── daily/               # Day-wise chunked episodic daily files
  │     │           ├── corpus-day-01.jsonl
  │     │           └── ...
  │     └── results/
  │           └── per-query/           # Default per-query evaluation summary & detail reports
  │
  └── adhd-diversified/               # Neurodivergent ADHD Profile (backyard astronomy + AI developer)
        ├── data/                     # Root data and graph files (12.8k records)
        │     ├── corpus.jsonl
        │     ├── corpus-biographical.jsonl
        │     ├── ...
        │     └── daily/               # Day-wise chunked episodic daily files
        │           ├── corpus-day-01.jsonl
        │           └── ...
        └── results/
              ├── per-query/           # Default per-query profile evaluation reports
              ├── hyperfocus/          # Forced HYPERFOCUS override evaluation reports
              ├── divergent/           # Forced DIVERGENT override evaluation reports
              └── systematizer/        # Forced SYSTEMATIZER override evaluation reports
```

---

## 📊 3-Way Cognitive Benchmark (1-Year Scale)

We evaluate retrieval quality across three experimental conditions:
1. **Baseline**: Pure vector similarity search (nearest-neighbor cosine/L2 distance), similar to traditional vector databases.
2. **Similarity**: The full Spector query pipeline (Bloom filter tag gating, valence checks, and active projects) but ranked via *pure* cosine similarity.
3. **Cognitive**: The full Spector pipeline, ranking candidates using **fused scoring** (similarity, importance, and temporal decay) and expanding results via **3-layer graph traversal** (Hebbian, Temporal, Entity).

Both datasets represent a **365-day (1-year)** continuous daily timeline of narrative experiences:
* **Balanced Baseline Scale**: 11,367 total memories.
* **ADHD Diversified Scale**: 12,879 total memories.

### I. Run 1: Balanced Baseline Dataset (Standard Family)
* **Baseline nDCG@10**: 0.110
* **Similarity nDCG@10**: 0.320
* **Cognitive nDCG@10**: 0.311 (**+182% lift over baseline**)
* **Cohen's d (vs Baseline)**: **0.694** (p = 0.0025, ⭐ Large effect size, highly significant)
* **Average Latency**: ~131 ms

### II. Run 2: ADHD Diversified Dataset (Neurodivergent Profiles)

Evaluating how specialized cognitive profiles perform under intense focus and highly associative narratives:

| Evaluation Profile | Baseline nDCG | Similarity nDCG | Cognitive nDCG | Cohen's d (vs Baseline) | p-value (vs Baseline) | Delta (vs Similarity nDCG) | Avg Latency |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| **`HYPERFOCUS`** | 0.091 | 0.293 | **0.545** | **1.454** | **<0.00001** | **+0.252** (Highly Significant!) | ~174 ms |
| **`BALANCED`** (Default) | 0.091 | 0.293 | **0.309** | **0.748** | **0.00045** | **+0.016** (Medium-large effect) | ~104 ms |
| **`DIVERGENT`** | 0.091 | 0.293 | **0.296** | **0.759** | **0.00037** | **+0.003** (Medium-large effect) | ~158 ms |
| **`SYSTEMATIZER`** | 0.091 | 0.293 | **0.270** | **0.689** | **0.00122** | **-0.023** (Medium-large effect) | ~111 ms |

* **HYPERFOCUS Advantage**: Routing searches through a `HYPERFOCUS` state (clamping temporal decay to zero for active focus topics) results in an absolute **+25.2% nDCG lift** over pure similarity and a **+500% lift** over raw vector search.
* **Gating Performance**: Pre-filtering with 64-bit Bloom filter **Synaptic Tag Gating** eliminates ~90% of candidates in 1 CPU cycle, preventing semantic noise and decay dilution over the 365-day timeline.

---

## 🧬 Memory Record Schema

Every entry in the `corpus.jsonl` follows the `BenchmarkCorpusRecord` layout:

```json
{
  "id": "episodic-day-142-turn-3",
  "timestampMs": 1782295628000,
  "text": "Finished testing the backyard computerized telescope mount. Alignment completed successfully on Polaris.",
  "memoryType": "EPISODIC",
  "synapticTags": ["backyard-astronomy", "telescope", "testing"],
  "interest": 0.9,
  "challenge": 0.8,
  "novelty": 0.7,
  "urgency": 0.2,
  "valence": 50,
  "arousal": 60
}
```

### Cognitive Metadata Definitions:
* **Interest, Challenge, Novelty, Urgency (ICNU)**: Ingestion signals that combine into the memory's structural *importance*, governing consolidate evictions.
* **Valence & Arousal**: Emotional coloring. Negative valence indicates issues/bugs; positive valence represents milestones. High arousal triggers "flashbulb memory" mechanics, slowing decay.
* **Synaptic Tags**: Hashed into an inline 64-bit Bloom filter inside the 64-byte off-heap record headers for zero-overhead candidate pre-filtering.

---

## 🔗 Licensing
This data is published under the **Apache License, Version 2.0**.
For engine implementation details, see [spector-search](https://github.com/spectrayan/spector).
