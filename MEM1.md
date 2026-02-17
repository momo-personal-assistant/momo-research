# MEM1: Learning to Synergize Memory and Reasoning for Efficient Long-Horizon Agents

[Paper]()

## Summary
MEM1 proposes an end-to-end reinforcement learning framework that enables Large Language Model agents to maintain constant memory usage while operating over long-horizon, multi-turn interactions. This framework, from researchers at SMART/MIT, demonstrates superior efficiency and competitive or improved performance on multi-objective QA and interactive navigation tasks.

## Problem
* Unbounded context growth in multi-turn LLM interactions leads to exploding computational costs and memory usage (O(N^2) or O(N)).
* LLMs trained on short contexts struggle to generalize to longer, out-of-distribution input lengths, causing performance degradation.
* Accumulating irrelevant information in the context overloads the model's attention, hindering reasoning and leading to the "lost in the middle" problem.

## Method
* Introduces a compact shared internal state () that iteratively consolidates prior memory with new observations, becoming the agent's only retained memory.
* Prunes all previous turn context, tool outputs, and observations after consolidation, maintaining constant memory usage across turns.
* Trains the agent end-to-end using reinforcement learning (PPO) where task success incentivizes the agent to learn memory consolidation intrinsically.

## Results
* Maintains an almost constant peak token count on multi-objective QA tasks while other methods show linear growth, achieving up to 72.9% reduction in peak tokens.
* Surpasses larger LLMs (e.g., Qwen2.5-14B-Instruct) in performance on complex 8- and 16-objective QA tasks, showing better generalization to long horizons.
* Achieves the highest average final reward (70.87) on WebShop navigation with significantly lower resource usage (2.8x lower peak tokens, 1.5x faster inference) compared to baselines.

## Takeaways
* Inference-time reasoning can be synergized with memory consolidation, allowing the agent to learn to extract and retain key information while discarding irrelevant details.
* Reinforcement learning, when combined with context pruning, naturally incentivizes the agent to develop effective internal memory management strategies.
* Composing existing single-objective datasets into multi-objective tasks creates effective long-horizon benchmarks for training and evaluating memory-efficient agents.