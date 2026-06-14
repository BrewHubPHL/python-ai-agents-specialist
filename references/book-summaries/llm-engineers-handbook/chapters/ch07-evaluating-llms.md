# Chapter 7: Evaluating LLMs

## Core Idea
LLM evaluation is multi-dimensional — no single metric suffices; combine general-purpose benchmarks (MMLU, IFEval, Chatbot Arena), domain-specific leaderboards, task-specific tests, and RAG-system evaluation that covers retrievers and post-processors, not just the model.

## Frameworks Introduced
- **Model-only evaluation**: Assess LLM capabilities without RAG or prompt engineering wrappers.
  - When to use: Picking base models, verifying fine-tuning improved (not degraded) capabilities.
  - How: Compare base vs. fine-tuned on MMLU, HellaSwag, IFEval, AlpacaEval, MT-Bench.
- **RAG system evaluation**: Evaluate retrieval + generation pipeline end-to-end.
  - When to use: Production GenAI apps where retrieval quality drives answer quality.
  - How: Test retriever recall, answer faithfulness to context, latency — not just perplexity.
- **Benchmark-as-signal**: Treat leaderboards as correlated signals, not ground truth — public benchmarks can be gamed.

## Key Concepts
- **MMLU**: 57-subject multiple-choice knowledge test.
- **IFEval**: Instruction-following with explicit constraints (e.g., no commas).
- **Chatbot Arena / AlpacaEval**: Human or automatic head-to-head preference for instruct models.
- **MT-Bench**: Multi-turn conversation coherence.
- **GAIA**: Agentic tool-use and multi-step reasoning.
- **Perplexity / training loss**: Pre-training monitoring metrics (lower perplexity = better).
- **Contamination detection**: Unlikely +10 MMLU jump after SFT signals benchmark leakage.
- **Hallucinations Leaderboard**: 16-task suite across QA, summarization, dialogue, fact-checking.

## Mental Models
- ML evaluation uses objective metrics (accuracy, F1); LLM evaluation needs qualitative + multi-benchmark consensus.
- Match benchmark to product: chatbot → Chatbot Arena; extraction → IFEval; medical → Open Medical-LLM Leaderboard.
- For RAG systems, a great LLM with bad retrieval still produces bad answers — evaluate the whole pipeline.

## Anti-patterns
- **Single benchmark shipping decisions**: One leaderboard score is insufficient — look for converging signals.
- **Ignoring fine-tuning regression**: Good SFT can degrade MMLU knowledge — always compare to base model.
- **Model-only eval for RAG products**: Testing the LLM in isolation misses retriever failure modes.

## Worked Example
**Fine-tuning evaluation workflow**:
1. Record base model MMLU, HellaSwag, IFEval scores
2. Fine-tune (SFT + DPO)
3. Re-run same benchmarks on fine-tuned checkpoint
4. Flag: MMLU drop > few points → possible overfitting or bad data; MMLU +10 → likely contamination
5. Add task-specific eval: generate LinkedIn posts, human-rate voice match
6. For RAG: measure faithfulness (answer supported by retrieved chunks?)

## Key Takeaways
1. LLM eval is inherently more subjective than classical ML — use multiple benchmarks.
2. Pre-training: monitor loss/perplexity; post-training: MMLU + IFEval + Arena-style prefs.
3. Domain leaderboards (medical, code, hallucination) target specialized deployments.
4. RAG evaluation must cover retriever, prompt assembly, and generation jointly.
5. Benchmarks are signals — watch for gaming, contamination, and human-eval bias toward long Markdown answers.

## Connects To
- **Ch 5–6**: Validate TwinLlama SFT and DPO checkpoints.
- **Ch 9**: RAG inference evaluation complements model eval.
- **Ch 4**: RAG faithfulness connects to "answer only from context" prompt design.
