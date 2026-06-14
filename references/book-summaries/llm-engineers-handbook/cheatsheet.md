# Cheatsheet — LLM Engineer's Handbook

## Architecture decisions

| Situation | Do | Because |
|---|---|---|
| New LLM product from scratch | FTI pipelines + feature store + model registry | Decouples data, training, serving; prevents training-serving skew |
| Adding a new data source | New crawler outputting existing category (article/repo/post) | Avoids rewriting downstream feature/training/inference code |
| Client needs personalized voice | Fine-tune on personal data + RAG over same corpus | SFT/DPO for style; RAG for factual grounding from past content |
| Generic ChatGPT is insufficient | Build LLM system (ETL → RAG → SFT → serve) | Chat UI can't automate data injection, versioning, or evaluation |

## Fine-tuning decisions

| Goal | Samples (heuristic) | Method |
|---|---|---|
| General-purpose instruct | ~1M high-quality pairs | SFT + preference alignment |
| Task-specific (summarize, translate) | 100–100K | SFT; few-shot prompting if large base model |
| Domain-specific (medicine, law) | Varies by domain size | SFT on domain corpus |
| Writing style / tone | 100–10K preference pairs | DPO after SFT |
| Limited GPU memory | Any | LoRA or QLoRA |

## RAG decisions

| Symptom | Fix |
|---|---|
| Hallucinations on private facts | RAG with "answer only from context" prompt |
| Stale knowledge (post-cutoff events) | RAG ingestion pipeline on fresh documents |
| Retrieval misses relevant docs | Query expansion + reranking |
| Results from wrong author | Self-querying with author_id metadata filter |
| Bad answers despite good LLM | Evaluate retriever separately — fix retrieval first |

## Inference decisions

| Constraint | Technique |
|---|---|
| Slow token generation | Speculative decoding + continuous batching |
| High KV memory | Quantization + shorter max context |
| Low GPU utilization | Continuous batching via vLLM/TGI |
| Need 2–4× forward pass speed | Static KV cache + torch.compile |

## Deployment decisions

| User experience | Deployment type |
|---|---|
| Chatbot, interactive writing | Online real-time (REST/WebSocket) |
| Summarize 1000 docs overnight | Offline batch transform |
| 5+ minute jobs, cost-sensitive | Asynchronous (queued) |
| Traffic spikes | Async queue or autoscaling real-time |

## Data quality gates (before training)

1. **Accuracy** — factually correct, relevant to instruction
2. **Diversity** — topics, lengths, styles represented
3. **Complexity** — multi-step reasoning, not trivial pairs
4. **Dedup** — no near-duplicate samples
5. **Decontaminate** — no benchmark test-set overlap

## Evaluation signals

| Observation | Likely cause |
|---|---|
| MMLU +10 after SFT | Benchmark contamination in training data |
| MMLU drop after SFT | Catastrophic forgetting or overfitting |
| Good benchmarks, bad product | Evaluating model only, not RAG system |
| Long Markdown answers win Arena | Human eval bias — don't over-optimize for length |

## MLOps triggers for retrain

- Model performance decay in production
- New labeled/crawled data available
- Distribution shift detected in monitoring
- Unhandled edge cases from user feedback

## LLM Twin MVP scope (minimum)

1. Crawl LinkedIn, Medium, Substack, GitHub
2. Fine-tune OSS LLM + populate vector DB
3. Generate LinkedIn posts (prompt + RAG)
4. Web UI for link config, collection trigger, prompting
