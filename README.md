## Introduction

**Pareto‑RAG** is a sentence–entity hybrid retrieval strategy that preserves **high recall under a strict token budget**. It separates (i) high‑recall, **entity‑driven document discovery** from (ii) high‑precision, **sentence‑level selection**, then chooses a **Pareto‑optimal subset** of sentences that maximizes evidence coverage per token. The pipeline plugs into any standard RAG stack: extract/normalize entities, (optionally) expand with short random walks on an entity–document graph, score at the sentence level, and solve a budgeted coverage problem to decide what to pass to the LLM.

---

## Motivation

Typical RAG systems over-fetch large chunks and later pay for them with expensive reranking/summarization—even though only a **handful of short sentences** usually support the final answer (especially in multi‑hop QA). Chunk‑level retrieval dilutes precision, increases latency/cost, and still misses rare or aliased entities. Pure dense sentence retrieval, on the other hand, often **loses global recall**.

**Pareto‑RAG addresses both sides:**

1. **Entity‑first recall** quickly jumps to the right documents (robust to aliasing via embedding similarity).
2. **Lightweight random walks** over the entity–document graph recover indirectly connected evidence.
3. **Sentence‑level scoring** ensures you only ship what’s necessary.
4. **Budgeted / Pareto selection** keeps the best recall–cost trade‑off within a fixed token budget.

---

## Key Ideas

1. **Entity‑first recall**  
   Extract entities from the question, map them to documents via an entity↔document edge table or graph DB, and optionally use embedding similarity for fuzzy/alias matching.

2. **Graph expansion (optional, for long‑tail recall)**  
   Run a small number of constrained random walks (or personalized PageRank–style propagation) on the entity–document graph to surface indirectly connected but relevant evidence.

3. **Sentence‑level scoring**  
   Split candidate chunks into sentences and score each with a hybrid objective:

   \[
   \text{score} = \lambda \cdot \text{semantic\_similarity} + (1 - \lambda) \cdot \text{entity\_overlap}
   \]

   (optionally add BM25 / TF‑IDF / cross‑encoder reranker terms).

4. **Pareto / budgeted selection**  
   Under a token budget **B**, select a subset of sentences that maximizes coverage/diversity/score while staying ≤ **B**. Implement as **greedy budgeted maximum coverage** or **knapsack**, pruning dominated solutions to approximate the Pareto frontier.

5. **Pluggable & model‑agnostic**  
   Works with any embedding/reranker model and any graph backend (NetworkX, FalkorDB, Neo4j, …).

6. **Multi‑hop‑aware metrics**  
   Report **one_found** (≥1 gold doc in top‑K) and **all_found** (all gold docs in top‑K) **plus token usage**, exposing the true recall–cost curve your production system will face.

