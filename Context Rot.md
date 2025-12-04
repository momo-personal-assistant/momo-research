# Context Rot Guide

> How Increasing Input Tokens Impacts LLM Performance

---

## 1. Background & Key Findings

### Why This Research

- **Problem**: NIAH (Needle in a Haystack) is the standard benchmark for long-context LLM performance
  - Hides info ("The secret code is 7392") in long documents, asks model to find it
  - Most models score near-perfect → "long-context is solved" belief
- **But NIAH only tests lexical matching** (question and answer share keywords)
  - Doesn't test semantic inference
  - Doesn't test filtering similar-but-wrong info
  - Doesn't test finding blended information
- **So Chroma tested 18 LLMs on realistic tasks** → discovered **Context Rot**

### Context Rot

> Performance degrades as input tokens increase—even on simple tasks.

- **Universal**: All 18 models affected
- **Simple tasks too**: Not just complex reasoning
- **Model-specific**: Claude, GPT, Gemini, Qwen each degrade differently

📊 **[CHART: Hero Plot]** - Performance comparison across Claude Sonnet 4, GPT-4.1, Qwen3-32B, Gemini 2.5 Flash

---

## 2. Key Terms

| Term | Definition |
|------|------------|
| **Needle** | Target information hidden in a long document |
| **Haystack** | Long background text (10k-100k+ tokens) |
| **Distractor** | Similar-but-incorrect information that confuses models |
| **Semantic similarity** | Meaning-based similarity (0-1 score via embeddings) |
| **Lexical matching** | Keyword-based matching without semantic understanding |

---

## 3. Six Experiments

### 3.1 Needle-Question Similarity

**Question**: Does semantic similarity between question and answer affect retrieval?

| Condition | Result |
|-----------|--------|
| Short context (~1k tokens) | High accuracy regardless of similarity |
| Long context (10k+ tokens) | Low-similarity pairs show sharp performance drop |

**Takeaway**: As context grows, semantic similarity between question and answer becomes critical.

📊 **[CHART: Needle-Question Similarity]** - Blue (high similarity) vs red (low similarity) across input lengths

---

### 3.2 Distractor Impact

**Question**: How do similar-but-wrong answers affect performance?

| Condition | Impact |
|-----------|--------|
| No distractors | Baseline performance |
| +1 distractor | Performance drops |
| +4 distractors | Compounded degradation |

**Distractor #3 (time change)** caused the most damage. **Claude** had lowest hallucination; **GPT** had highest.

📊 **[CHART: Distractor Impact]** - Performance by distractor count and individual distractor effects

---

### 3.3 Needle-Haystack Similarity

**Question**: Does the needle "blending in" with surrounding text hurt performance?

| Haystack | Needle | Performance |
|----------|--------|-------------|
| PG Essays | PG (high similarity) | Lower |
| PG Essays | arXiv (low similarity) | **Higher** |

**Takeaway**: When needle differs from background, it "stands out" and is easier to find—but pattern is non-uniform.

📊 **[CHART: Needle-Haystack Similarity]** - Cross-domain performance comparison

---

### 3.4 Haystack Structure

**Question**: Is it easier to find info in logically organized text?

| Structure | Performance |
|-----------|-------------|
| Original (coherent) | Lower |
| Shuffled (random order) | **Higher** |

**Counterintuitive**: Shuffled text yields better results across all 18 models. Hypothesis: coherent text disperses attention; shuffled text makes needle stand out.

📊 **[CHART: Haystack Structure]** - Original vs Shuffled performance

---

### 3.5 Repeated Words Task

**Question**: Can models accurately replicate long sequences?

Input: "apple apple apple BANANA apple apple..." → Replicate exactly.

| Length | Result |
|--------|--------|
| Short (25-100 words) | High accuracy |
| Long (5k-10k words) | Sharp degradation, random word generation |

**Position matters**: Unique word at beginning → higher accuracy.

📊 **[CHART: Repeated Words]** - Performance by model family across input lengths

---

### 3.6 LongMemEval

**Question**: Can models find answers in long conversation histories?

| Condition | Tokens | Performance |
|-----------|--------|-------------|
| Focused | ~300 | High accuracy |
| Full | ~113k | Significant degradation |

**Claude** showed largest gap (abstains when uncertain). **GPT/Gemini/Qwen** degrade but still attempt answers.

📊 **[CHART: LongMemEval]** - Focused vs Full performance by model family

---

## 4. Model Behavior Summary

| Model | Hallucination | When Uncertain | Notes |
|-------|---------------|----------------|-------|
| **Claude** | Lowest | States "not found" | Most conservative |
| **GPT** | Highest | Answers confidently | Even when wrong |
| **Gemini** | Medium | Random generation | Starts at 500-750 words |
| **Qwen** | Medium | Doesn't attempt | Random output after 5k words |

---

## 5. Practical Guidelines

### Core Principle

```
Information EXISTS in context ≠ Performance guaranteed
WHERE and HOW it's placed = Determines performance
```

### Recommendations

- [ ] **Place important info early** in context
- [ ] **Minimize distractors** (similar-but-wrong content)
- [ ] **Monitor context length** — all models degrade
- [ ] **Choose model wisely**: Claude for reliability, expect degradation for long contexts

### Model Selection

| Need | Recommendation |
|------|----------------|
| Low hallucination | Claude |
| Long context required | All degrade — use context engineering |
| Uncertainty expression | Claude (explicitly abstains) |

---

## 6. Limitations & Future Work

**Limitations**:
- Tests focused on simple retrieval tasks
- Real-world tasks (synthesis, multi-step reasoning) likely show worse degradation
- Shuffled > coherent result remains unexplained

**Future Research**:
- Mechanistic interpretability of attention patterns
- More realistic benchmarks
- Disentangling complexity vs. length effects

---

## References

```bibtex
@techreport{hong2025context,
  title   = {Context Rot: How Increasing Input Tokens Impacts LLM Performance},
  author  = {Hong, Kelly and Troynikov, Anton and Huber, Jeff},
  year    = {2025},
  month   = {July},
  institution = {Chroma}
}
```

*Source: [Chroma Research](https://research.trychroma.com/context-rot)*
