# EXP-09 v3 — Embedding Benchmark Summary Report

**Generated:** 2026-04-23 15:59
**Dataset:** arXiv Quantitative Finance (q-fin) — Full-text research papers from [arXiv.org](https://arxiv.org/list/q-fin/recent)
**Corpus:** 5 topic documents → 10009 chunks (SEMANTIC chunking · target 300 chars · 60 char overlap)
**Questions:** 31 evaluation queries with ground-truth source documents

---

## 1. Model Rankings

| Rank | Model | Recall@1 | Recall@3 | Recall@5 | MRR | Fallback? |
|------|-------|----------|----------|----------|-----|-----------|
| 🥇 #1 | **financial-news-distilroberta (Finance domain) [fallback: all-mpnet-base-v2]** | 67.7% | 100.0% | 100.0% | **0.833** | → `all-mpnet-base-v2` |
| 🥈 #2 | **all-MiniLM-L6-v2 (General baseline)** | 67.7% | 100.0% | 100.0% | **0.828** | — |
| 🥉 #3 | **BGE-small-en-v1.5 (General compact)** | 64.5% | 96.8% | 100.0% | **0.793** | — |
|    #4 | **FinBERT-tone (Finance sentiment)** | 48.4% | 90.3% | 96.8% | **0.683** | — |

> **Primary metric: MRR** (Mean Reciprocal Rank). Rewards rank-1 hits more than rank-5.
> **Recall@3** is the practical RAG window — most pipelines send top-3 chunks to the LLM.

---

## 2. Dataset & Corpus

**Source:** arXiv Quantitative Finance (q-fin) Research Papers — Full-text academic papers
covering banking, portfolio management, risk management, derivatives, and other finance topics.

**Corpus construction:**
1. Downloaded ~40 full-text research papers from arXiv q-fin category.
2. Papers were classified into finance topic categories using keyword matching.
3. Papers on the same topic were combined into thematic documents.
4. Each document was chunked using the **SEMANTIC** strategy.

| Topic | Papers | Chunks |
|-------|--------|--------|
| DOC-01 Banking & Credit Risk | 3 | 692 |
| DOC-02 Portfolio Management & Asset Allocation | 4 | 1208 |
| DOC-04 Derivatives & Options Pricing | 6 | 1163 |
| DOC-05 Risk Management & VaR | 5 | 2165 |
| DOC-08 Machine Learning in Finance | 16 | 4781 |

---

## 3. Chunking Strategy

**Strategy used: SEMANTIC**

| Strategy | Description | When to prefer |
|----------|-------------|----------------|
| **Recursive** | Splits on `\n\n` → `\n` → `.` → space → char; fixed window with overlap | Speed, reproducibility |
| **Semantic** | Sentence boundary detection + cosine similarity gate; new chunk when similarity < threshold | Quality, coherence |

- Total chunks: **10009** from 5 documents
- Avg chunk size: **174 chars**
- Semantic threshold: 0.75 (cosine sim below this starts a new chunk)

**Why use full research papers:** Full academic papers provide comprehensive, detailed
content covering methodology, results, and analysis - ideal for testing RAG retrieval
on complex, technical finance topics.

**Why v2 had no chunking:** v2 embedded whole documents as single vectors.
With long finance docs, a single vector averages over many sub-topics and loses
specificity. Chunking creates granular, semantically focused embeddings that
match narrower question intents far more precisely.

---

## 4. Domain vs General Embedding Models

**General models are competitive** (avg MRR: general=0.810 vs domain=0.758)

This can occur when: (1) domain model was loaded via fallback and the fallback model
was not fine-tuned on finance-similarity tasks; (2) the corpus sentences are drawn from
news wire text, which general models have seen extensively; (3) the query set tests
surface-level matching rather than deep domain semantics.

**Queries where domain models win most:**

- **Q14** Δ=+0.583 (domain=1.000 vs general=0.417)
  *What role do derivatives play in portfolio hedging strategies?*
- **Q49** Δ=+0.333 (domain=1.000 vs general=0.667)
  *How do monetary policy decisions affect banking operations?*
- **Q02** Δ=+0.333 (domain=0.667 vs general=0.333)
  *How do Basel regulations impact bank capital requirements?*
- **Q10** Δ=+0.250 (domain=0.750 vs general=0.500)
  *Explain Conditional Value at Risk (CVaR) and how it differs from VaR.*
- **Q21** Δ=+0.250 (domain=1.000 vs general=0.750)
  *How do credit risk, market risk, and operational risk interact in banking?*

**Queries where general models win most:**

- **Q11** Δ=-0.583 (general=1.000 vs domain=0.417)
  *How do credit risk models relate to portfolio optimization strategies?*
- **Q15** Δ=-0.500 (general=1.000 vs domain=0.500)
  *How does market liquidity affect derivative pricing?*
- **Q04** Δ=-0.375 (general=1.000 vs domain=0.625)
  *What is the Sharpe ratio and how is it used in portfolio evaluation?*
- **Q12** Δ=-0.375 (general=0.750 vs domain=0.375)
  *What is the relationship between banking regulation and systemic risk?*
- **Q08** Δ=-0.333 (general=1.000 vs domain=0.667)
  *What are the Greeks in options trading and why are they important?*

---

## 5. Fallback Model Notes

- **D_FinNews** (`nickmuchi/financial-news-distilroberta-base`): primary unavailable → fell back to `sentence-transformers/all-mpnet-base-v2`

---

## 6. Recommendations

1. **Best model for production:** financial-news-distilroberta (Finance domain) [fallback: all-mpnet-base-v2] (MRR = 0.833)
2. **Chunking:** Apply SEMANTIC chunking before indexing — it yields more precise chunk-level retrieval than whole-document embedding.
3. **Corpus quality matters:** Replacing hand-crafted demo documents with real financial
   news sentences improves retrieval realism and benchmark validity.
4. **FinBERT note:** `yiyanghkust/finbert-tone` is a _sentiment classifier_, not a
   sentence similarity model. Its embeddings may underperform on retrieval tasks.
   Consider `nickmuchi/financial-news-distilroberta-base` as primary finance model.
5. **Scale up:** For production RAG, combine the best embedding model with a
   bi-encoder → cross-encoder reranking pipeline for maximum precision.

---

## 7. Output Files

```
exp09_outputs_v3/
├── a_minilm_results.txt / .json
├── b_bge_results.txt / .json
├── c_finbert_results.txt / .json
├── d_finnews_results.txt / .json
├── exp09v3_bar_metrics.png       ← 4-panel bar chart
├── exp09v3_per_question_rr.png   ← per-question line chart
├── exp09v3_domain_delta.png      ← domain vs general delta
└── exp09v3_summary_report.md     ← this report
```