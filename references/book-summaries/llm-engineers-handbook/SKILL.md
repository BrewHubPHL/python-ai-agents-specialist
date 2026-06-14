---
name: llm-engineers-handbook
description: "Knowledge base from \"LLM Engineer's Handbook\" by Paul Iusztin & Maxime Labonne. Use when applying FTI pipeline architecture, building RAG systems, fine-tuning LLMs (SFT/DPO/LoRA), optimizing inference, deploying LLM services, or studying LLMOps for production GenAI."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# LLM Engineer's Handbook
**Author**: Paul Iusztin & Maxime Labonne | **Pages**: ~523 | **Chapters**: 11 | **Generated**: 2025-06-14

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `RAG`, `DPO`, `FTI`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch05`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### LLM Twin — the book's end-to-end project
Use when building a personalized GenAI product, not just calling an API. An **LLM Twin** is an AI character that projects your writing style, voice, and personality into an LLM through fine-tuning on your digital data plus RAG over your embeddings. It is a *writing co-pilot*, not a full digital clone — it reflects only the side of you present in training data. Prefer building an LLM **system** (automated data collection, preprocessing, storage, fine-tuning, RAG, evaluation) over using ChatGPT's web UI when you need persistent voice, grounding, and reproducibility.

### FTI Pipeline Architecture
Use when structuring any production ML/LLM application. Decompose into three logical pipelines with stable interfaces:
- **Feature pipeline**: raw data → features/labels → **feature store**
- **Training pipeline**: features → trained model → **model registry**
- **Inference pipeline**: features + model → predictions

Prefer FTI over monolithic batch pipelines (couples everything, prevents scaling) and stateless real-time (forces clients to pass feature state). The feature store eliminates **training-serving skew** by versioning features used identically at train and inference time. Each pipeline can use different tech stacks, teams, and hardware. FTI pipelines are *logical layers* — internal complexity is fine as long as store/registry interfaces stay stable.

### MVP scoping for LLM products
Use when starting with limited team/resources. An MVP must be **viable** (complete user journey, no half-features), not merely minimal. For LLM Twin: crawl personal content → fine-tune OSS model → populate vector DB → generate social posts via simple UI. Maximize value per engineering effort; build modular and scalable even in v1.

### Vanilla RAG (Retrieval-Augmented Generation)
Use when LLMs need private, fresh, or domain data without constant fine-tuning. Three independent modules:
1. **Ingestion** (batch): extract → clean → chunk → embed → load vector DB
2. **Retrieval** (online): embed query → similarity search (cosine distance) → top-K chunks
3. **Generation**: system + prompt template + context + LLM

RAG solves **hallucinations** (context is source of truth) and **stale/private data** (inject without retraining). Critical rule: preprocess user queries identically to ingestion — same cleaning, chunking, embedding model — or distances are meaningless.

### Advanced RAG retrieval stack
Use in production when naive single-vector search under-retrieves. Pipeline: **query expansion** (LLM generates N query variants) → **self-querying** (extract metadata filters like author_id) → **filtered vector search** (N searches, K results each) → **reranking** (cross-encoder scores against original query) → top-K → prompt → LLM. Most RAG engineering lives in retrieval, not the LLM call.

### Supervised Fine-Tuning (SFT)
Use after pre-training to teach chat format and task/domain specialization. Build **instruction datasets** (instruction + answer pairs). Quality dimensions: **accuracy**, **diversity**, **complexity** — filter via rules (length, keywords, format), deduplication, and benchmark decontamination. Sample heuristics: ~1M for general-purpose instruct; 100–100K for task-specific; fewer for large (70B) models with high-quality data (LIMA). Use **LoRA/QLoRA** when GPU memory is limited.

### Direct Preference Optimization (DPO)
Use when SFT cannot capture subjective preferences (tone, naturalness, engagement). Format: instruction + **chosen** answer + **rejected** answer. DPO is less destructive than SFT and sample-efficient for style alignment (100–10K pairs for task-specific). Generate preferences via LLM diversity (multi-model, temperature variation); evaluate with humans or stronger LLMs. Run DPO on SFT checkpoint — book produces TwinLlama-3.1-8B-DPO.

### Inference optimization
Use when serving decoder-only LLMs in production. Bottleneck is sequential token generation. Stack: **KV cache** (mandatory) → **static KV cache** + `torch.compile` (up to 4× forward) → **continuous batching** (no straggler wait) → **speculative decoding** (draft model predicts N tokens, target validates) → **quantization** → production engines (TGI, vLLM, TensorRT-LLM). Draft and target models must share tokenizers.

### Deployment architecture selection
Use before serving models. Four criteria: **throughput** (RPS), **latency** (ms), **data** (format/volume), **infrastructure** (GPU/network). Three types: **online real-time** (chatbots, REST/WebSocket), **asynchronous** (queued, long jobs, spike absorption), **offline batch** (overnight predictions). Latency and throughput trade off — batching helps throughput but can hurt per-request latency. LLM Twin uses online real-time: FastAPI + SageMaker endpoint with autoscaling.

### MLOps → LLMOps progression
Use when operationalizing LLM applications. **DevOps** automates code ship; **MLOps** adds data and model as first-class citizens (model registry, feature store, orchestrator, experiment tracking); **LLMOps** adds prompt versioning/monitoring, guardrails, feedback loops. Six MLOps principles: automation, versioning, experiment tracking, testing (code+data+model), monitoring, reproducibility. Implement CI/CD (GitHub Actions) + CT (ZenML retrain triggers) + prompt monitoring (Opik). Data changes can trigger retrains without code changes — unlike pure DevOps.

### Data-centric, model-agnostic design
Use as default architecture principle. Success depends more on data pipeline quality than which base LLM you pick. Build systems where any programmatically controllable LLM (OpenAI API, Llama, etc.) can be swapped without rewriting data/training/serving infrastructure.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-llm-twin-concept-architecture.md) | Understanding the LLM Twin Concept and Architecture | LLM Twin, MVP, FTI architecture, training-serving skew |
| [ch02](chapters/ch02-tooling-installation.md) | Tooling and Installation | Poetry, ZenML, Hugging Face, MongoDB, Qdrant, SageMaker |
| [ch03](chapters/ch03-data-engineering.md) | Data Engineering | ETL, crawler dispatcher, document categories, ODM |
| [ch04](chapters/ch04-rag-feature-pipeline.md) | RAG Feature Pipeline | Vanilla RAG, embeddings, chunking, ingestion pipeline |
| [ch05](chapters/ch05-supervised-fine-tuning.md) | Supervised Fine-Tuning | Instruction datasets, LoRA, QLoRA, data curation |
| [ch06](chapters/ch06-preference-alignment-dpo.md) | Fine-Tuning with Preference Alignment | DPO, preference datasets, chosen/rejected pairs |
| [ch07](chapters/ch07-evaluating-llms.md) | Evaluating LLMs | MMLU, IFEval, RAG evaluation, benchmark contamination |
| [ch08](chapters/ch08-inference-optimization.md) | Inference Optimization | KV cache, speculative decoding, quantization, vLLM |
| [ch09](chapters/ch09-rag-inference-pipeline.md) | RAG Inference Pipeline | Query expansion, self-querying, reranking |
| [ch10](chapters/ch10-inference-deployment.md) | Inference Pipeline Deployment | Real-time/async/batch, FastAPI, SageMaker autoscaling |
| [ch11](chapters/ch11-mlops-llmops.md) | MLOps and LLMOps | CI/CD, CT, Opik monitoring, DevOps→MLOps→LLMOps |

## Topic Index

- **Advanced RAG** → ch04, ch09
- **Autoscaling** → ch10
- **Continuous batching** → ch08
- **Cosine distance** → ch04
- **Crawler dispatcher** → ch03
- **Cross-encoder reranking** → ch09
- **Data decontamination** → ch05
- **DPO** → ch06
- **Embeddings** → ch04
- **ETL** → ch03
- **FastAPI deployment** → ch10
- **Feature store** → ch01, ch04
- **FTI architecture** → ch01
- **GitHub Actions CI/CD** → ch11
- **Hallucinations** → ch04
- **Hugging Face** → ch02, ch05
- **IFEval** → ch07
- **Inference optimization** → ch08
- **Instruction datasets** → ch05
- **KV cache** → ch08
- **LLM Twin** → ch01
- **LLMOps** → ch11
- **LoRA / QLoRA** → ch05
- **MMLU** → ch07
- **Model registry** → ch01, ch02
- **MongoDB data warehouse** → ch03
- **MVP** → ch01
- **Opik prompt monitoring** → ch02, ch11
- **Preference alignment** → ch06
- **Qdrant** → ch02, ch04
- **Query expansion** → ch09
- **RAG** → ch04, ch09
- **Reranking** → ch09
- **SageMaker** → ch02, ch10
- **Self-querying** → ch09
- **SFT** → ch05
- **Speculative decoding** → ch08
- **Training-serving skew** → ch01, ch04
- **Vanilla RAG** → ch04
- **ZenML** → ch02, ch03, ch11

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond this book, check related skills
or ask the agent directly.
