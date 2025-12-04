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


---

## 2. Key Terms

| Term | Definition | Example |
|------|------------|---------|
| **Needle** | Target information hidden in a long document | "The best writing advice from my college professor was 'write every day'" |
| **Haystack** | Long background text (10k-100k+ tokens) | Paul Graham essays, arXiv papers |
| **Distractor** | Similar-but-incorrect information that confuses models | "The worst advice from my high school friend was 'don't write'" |
| **Semantic similarity** | Meaning-based similarity (0-1 score via embedding cosine similarity) | "writing advice" and "composition tips" = high similarity |
| **Lexical matching** | Keyword-based matching without semantic understanding | Q: "What's the magic number?" → Find "The magic number is 42" |

---

## 3. Experiment Overview

| Experiment | What it tests | Key finding |
|------------|---------------|-------------|
| Needle-Question Similarity | Does semantic similarity between question and answer affect performance? | Lower similarity → sharp performance drop in long contexts |
| Distractor Impact | How do similar-but-wrong answers affect performance? | More distractors → worse performance, #2 and #3 most damaging |
| Needle-Haystack Similarity | Does the needle "blending in" hurt performance? | Different topic from background → actually better performance |
| Haystack Structure | Organized text vs shuffled text | **Paradox**: Shuffled text performs better |


---

## 4. Experiment Details

### 4.1 Needle-Question Similarity
![Needle-Question Similarity](https://research.trychroma.com/img/context_rot/longmemeval/needle_question_sim_arxiv.png)

> **TL;DR**: When asking "What's the writing advice?", if the answer contains "writing" it's easy to find, but if it says "composition tips" it's harder. This experiment measures how the semantic similarity between question and answer affects performance as context length increases.

**Experiment Design**

- **Hypothesis**:
  - Semantically similar question-answer pairs should be easier to find
  - If short context works fine, does longer context maintain the same performance?

- **Method**:
  1. Extract question-answer pairs from diverse texts (Paul Graham essays, arXiv papers)
  2. Measure semantic similarity of each pair using 5 embedding models (OpenAI, Jina, Voyage, etc.)
  3. Group into high-similarity pairs (0.7-0.8) and low-similarity pairs (0.4-0.5)
  4. Place same questions in haystacks of varying lengths, ask model to find answer
  5. Compare accuracy by input length

- **Similarity Diversification Method**:
  - Reduce embedding vectors with UMAP (compress high-dimensional vectors to 2D/3D, clustering similar items)
  - Cluster with HDBSCAN (separate clustered items into groups)
  - This systematically captures "high similarity" and "low similarity" groups

- **Measured Similarity Ranges**:
  | Data Source | Similarity Range | Meaning |
  |-------------|------------------|---------|
  | PG Essays | 0.445 ~ 0.775 | Low (0.4s) to high (0.7s) distribution |
  | arXiv | 0.521 ~ 0.829 | Generally higher similarity |

**Key Findings**

- Short context (~1k tokens): High accuracy regardless of similarity
- Long context (10k+ tokens): Sharp performance drop for low-similarity pairs
- **Conclusion**: As context grows, semantic similarity between question and answer significantly impacts performance
- Note: Needle position (front/middle/back) has minimal effect (tested 11 positions)


---

### 4.2 Distractor Impact Analysis

> **TL;DR**: When similar-but-wrong information is mixed into the context, models are more likely to select wrong answers. This experiment measures how performance degrades based on the number of "confusing wrong answers."

**Experiment Design**

- **Hypothesis**:
  - Models find answers easily when only correct info exists, but get confused with similar wrong answers
  - More wrong answers = more confusion?
  - Which type of wrong answer causes the most confusion?

- **Method**:
- ![distractors_var](https://research.trychroma.com/img/context_rot/niah/distractors_var.png)
  1. Create 4 distractors for each needle
  2. Distractors have similar topic/format but different core content
  3. Test under three conditions:
     - **Baseline**: Only needle in haystack
     - **Single**: Needle + 1 distractor
     - **Multiple**: Needle + all 4 distractors
  4. Measure accuracy for each condition

- **Distractor Design**:
  - Needle: "The best writing advice from my college professor was 'write every day'"
  - Distractor #1: Source change → "Advice from my college **classmate** was..."
  - Distractor #2: Sentiment change → "The **worst** advice was..."
  - Distractor #3: Time change → "During **high school**..."
  - Distractor #4: Certainty change → "Maybe...I think it was..."

**Key Findings**

- Even 1 distractor causes performance drop from baseline
- 4 distractors cause compounded degradation
- **Non-uniform distractor impact**:
  | Distractor | Impact | Notes |
  |------------|--------|-------|
  | #1 (Source change) | Medium | Standard impact |
  | #2 (Sentiment change) | High | Models often output this as correct answer |
  | #3 (Time change) | **Highest** | Causes largest performance drop |
  | #4 (Certainty change) | Medium | Standard impact |

- **Model-specific responses**:
  - Claude: States "cannot find answer" when uncertain → lowest hallucination
  - GPT: Outputs wrong answers confidently → highest hallucination


---

### 4.3 Needle-Haystack Similarity

> **TL;DR**: If the answer blends in with surrounding text (same topic), is it harder to find? If it's different, does it "stand out" and become easier to find? This experiment measures how semantic similarity between needle and background text affects performance.

**Experiment Design**

- **Hypothesis**:
  - Hiding writing-related answer in writing essays = too similar, hard to find?
  - Hiding ML-related answer in writing essays = stands out, easy to find?

- **Method**:
- ![needle_haystack](https://research.trychroma.com/img/context_rot/niah/needle_haystack.png)
  1. Prepare two types of haystack:
     - **PG Essays**: Paul Graham's writing/startup essays
     - **arXiv**: Information retrieval/ML academic papers
  2. Prepare two types of needle:
     - **PG needle**: Writing/startup related info
     - **arXiv needle**: Academic/technical info
  3. Test 4 combinations:
     - PG haystack + PG needle (high similarity: 0.529)
     - PG haystack + arXiv needle (low similarity: 0.368)
     - arXiv haystack + arXiv needle (high similarity: 0.654)
     - arXiv haystack + PG needle (low similarity: 0.394)
  4. Compare accuracy for each combination

**Key Findings**

- **In PG haystack**: arXiv needle (low similarity) performs **better** than PG needle (high similarity)
  - **Interpretation**: When needle differs from background, it "stands out" for easier identification
- **In arXiv haystack**: Almost no performance difference between needle types
- **Conclusion**: Non-uniform pattern varies by situation. Cannot simply generalize "lower similarity = always better"


---

### 4.4 Haystack Structure (Coherent vs Shuffled)

> **TL;DR**: Is it easier to find info in logically organized text? Or does shuffled sentence order make the answer stand out? This experiment measures how structural coherence affects performance.

**Experiment Design**

- **Hypothesis**:
  - Common sense suggests organized text is easier to search
  - But do LLMs follow the same pattern?

- **Method**:
- ![nh2](https://research.trychroma.com/img/context_rot/niah/nh2.png)
  1. Create two versions from same text:
     - **Original**: As-is, paragraphs/sentences in logical order
     - **Shuffled**: Same sentences randomly rearranged (same content, different order)
  2. Insert same needle into both versions
  3. Compare accuracy across 18 models

**Key Findings (Paradoxical!)**

- **Expected**: Better performance in logically organized text (Original)
- **Actual**: **Better performance in shuffled text**
- Consistent pattern across all 18 models and all needle-haystack combinations

**Why? (Hypotheses)**

- Logical flow may disperse model's attention across multiple sentences
- Shuffled sentences are recognized independently → needle stands out more
- Effect becomes more pronounced as context grows
- Exact cause not yet determined (needs further research)


---

### 4.5 Repeated Words Task

> **TL;DR**: Given a sequence like "apple apple apple BANANA apple apple...", ask the model to replicate it exactly. This experiment measures how accurately models can replicate as input length increases.

**Experiment Design**

- **Hypothesis**:
  - Does even simple replication become harder with longer input?
  - Does accuracy differ when unique word is at front vs back?

- **Method**:
  1. Generate input:
     - Example: "apple apple apple apple BANANA apple apple apple..."
     - Same word (apple) repeated with 1 unique word (BANANA) inserted
  2. Instruct model to "replicate this sequence exactly"
  3. Test various lengths (25-10,000 words) and 7 word combinations
  4. Compare output to input for accuracy (using Levenshtein distance)

- **Test Conditions**:
  - 12 length steps: 25 → 50 → 75 → 100 → 250 → 500 → 750 → 1,000 → 2,500 → 5,000 → 7,500 → 10,000 words
  - 7 word combinations:
    - apple/apples (singular/plural)
    - golden/Golden (case difference)
    - orange/run (completely different words)

**Key Findings**

- **All models show consistent length ↑ → performance ↓ pattern**
- Unique word at sequence front → higher accuracy (especially in long contexts)
- In long contexts, models start generating random words not in input

- **Model-specific behaviors**:
  | Model | Notable behavior |
  |-------|------------------|
  | Claude 3.5 Sonnet | Better performance than newer Claude models up to 8,192 tokens |
  | Claude Opus 4 | Slowest performance degradation. But 2.89% task refusal |
  | GPT-4.1 | 2.55% task refusal around 2,500 words |

---

### 4.6 LongMemEval

> **TL;DR**: Answer questions from long conversation history (~110k tokens). Models answer well when given only relevant info, but how much does performance drop when irrelevant conversations are mixed in?

**Experiment Design**

- **Hypothesis**:
  - If information exists, will models find it well?
  - Or does irrelevant information interfere?

- **Method**:
  1. Extract 306 questions from LongMemEval_s benchmark
  2. Test each question under two conditions:
     - **Full (~113,000 tokens)**: Entire conversation history (mostly irrelevant)
     - **Focused (~300 tokens)**: Only information needed to answer
  3. Compare accuracy between conditions

- **Question Types (3)**:
  - **Knowledge Update**: "First said A, later changed to B. What's the latest?"
  - **Temporal Reasoning**: "Did X first, then Y. What's the order?"
  - **Multi-session**: "Combining conclusions from last week and this week's meetings?"

**Key Findings**

- **Focused condition**: All models show high accuracy
- **Full condition**: Significant performance drop
- **Conclusion**: Even when needed info exists, excessive irrelevant context hurts performance

- **Model-specific patterns**:
  - **Claude**: **Largest** gap between Focused↔Full
    - Reason: Tends to abstain ("cannot find answer") when uncertain
    - Full condition: uncertainty ↑ → abstain ↑ → score ↓
  - **GPT, Gemini, Qwen**: Good in Focused, degraded in Full (smaller gap than Claude)

- **Difficulty by question type**:
  | Mode | Easy → Hard |
  |------|-------------|
  | Non-thinking | Knowledge Update > Multi-session > Temporal Reasoning |
  | Thinking mode | Knowledge Update > Temporal Reasoning > Multi-session |


---

## 5. Model Behavior Summary

| Model | Hallucination | When Uncertain | Notable behavior |
|-------|---------------|----------------|------------------|
| **Claude** | ✅ Lowest | States "cannot find answer" | Opus 4: 2.89% task refusal |
| **GPT** | ❌ Highest | Answers confidently even when wrong | GPT-3.5: 60% excluded by content filter |
| **Gemini** | Medium | Starts generating random words | Issues begin at 500-750 words |
| **Qwen** | Medium | Doesn't attempt task | Random output after 5,000 words |

---

## 6. Methodology

- **Experiment Design**
  - 8 input lengths × 11 needle positions tested
  - Temperature=0 for deterministic output (some model exceptions)
  - Qwen models use YaRN extension: 32k → 131k tokens

- **Evaluation**
  - LLM judge: calibrated to 99%+ agreement with human judgment
  - Manual labeling verification: NIAH ~500, LongMemEval ~600
  - Overall task refusal rate: 0.035% (69 out of 194,480 calls)

---

## 7. Practical Guidelines

### Core Principle

```
Information EXISTS in context ≠ Performance guaranteed
WHERE and HOW it's placed = Determines performance
```

### Recommendations

- [ ] **Place important info early** in sequence — earlier unique words = higher accuracy
- [ ] **Minimize distractors** (similar-but-wrong info) — even 1 causes performance drop
- [ ] **Monitor context length** — all models degrade as length increases
- [ ] **Choose model based on characteristics**
  - Reliability/accuracy priority → Claude (lowest hallucination)
  - Long context unavoidable → expect degradation, strengthen context engineering

### Model Selection Guide

| Requirement | Recommended | Reason |
|-------------|-------------|--------|
| Low hallucination | Claude | Most conservative, abstains when uncertain |
| Long context required | All degrade | Compensate with context engineering |
| Need uncertainty expression | Claude | Explicitly states "no answer found" |

---

## 8. Limitations

- **Test scope limitations**
  - This research focused on relatively simple information retrieval tasks
  - Real-world use requires synthesis and multi-step reasoning
  - **Actual degradation likely > these experiment results**

- **Unexplained phenomena**
  - Why does performance drop in logically organized text?
  - How does attention mechanism respond to structural coherence?
  - How to separate task complexity from context length effects?

---

## 9. Future Research

1. **Mechanistic Interpretability**
   - Analyze internal workings to understand why certain structural patterns affect performance

2. **Attention Mechanism Analysis**
   - Research how input structure affects attention distribution

3. **More Realistic Benchmarks**
   - Complex task evaluation beyond current narrow-scope assessments

4. **Complexity vs Length Separation**
   - Separately measure task difficulty vs context length degradation

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

---

*Source: [Chroma Research](https://research.trychroma.com/context-rot)*
