# Glossary — LLM Engineer's Handbook

**Alpaca template** — Instruction dataset format with system, instruction, input, and output fields (Ch 5)

**Asynchronous inference** — Queue-based model serving where clients don't wait synchronously (Ch 10)

**Chosen answer** — Preferred response in a DPO preference pair (Ch 6)

**Continuous batching** — Replace completed inference requests immediately in GPU batch (Ch 8)

**Cosine distance** — Similarity metric for comparing embedding vectors in RAG retrieval (Ch 4)

**Cross-encoder reranking** — Neural model scoring query–chunk relevance after initial vector search (Ch 9)

**CT (Continuous Training)** — Automated model retraining triggered by data or performance changes (Ch 11)

**Data category** — Document type abstraction: article, repository, or post (Ch 3)

**Direct Preference Optimization (DPO)** — Preference alignment without a separate reward model (Ch 6)

**EmbeddedQuery** — Query domain object with embedding vector for vector search (Ch 9)

**Embeddings** — Dense numerical vectors encoding semantic meaning of text/objects (Ch 4)

**ETL (Extract, Transform, Load)** — Data pipeline pattern for crawling and warehousing raw content (Ch 3)

**Feature pipeline** — FTI component: raw data → features/labels → feature store (Ch 1, Ch 4)

**Feature store** — Versioned storage for ML features shared by training and inference (Ch 1)

**FTI architecture** — Feature/Training/Inference pipeline decomposition for ML systems (Ch 1)

**Hallucination** — LLM generating confident but false information without grounding (Ch 4)

**Hugging Face Hub** — Model registry and dataset hosting platform used in LLM Twin (Ch 2)

**IFEval** — Benchmark testing instruction-following with explicit constraints (Ch 7)

**Inference pipeline** — FTI component: features + model → predictions (Ch 1, Ch 9–10)

**KV cache** — Cached attention key-value pairs to avoid recomputation during generation (Ch 8)

**LLM Twin** — AI character mimicking a person's writing style via fine-tuning + RAG (Ch 1)

**LLMOps** — MLOps practices specialized for LLM prompt monitoring, guardrails, and scale (Ch 11)

**LoRA** — Low-Rank Adaptation for parameter-efficient fine-tuning (Ch 5)

**MMLU** — Massive Multi-Task Language Understanding knowledge benchmark (Ch 7)

**Model registry** — Versioned storage for trained models with training metadata (Ch 1, Ch 2)

**MVP** — Minimum Viable Product with complete end-to-end user journey (Ch 1)

**ODM** — Object Document Mapper for MongoDB document classes (Ch 3)

**Opik** — Comet ML prompt monitoring tool for LLMOps (Ch 2, Ch 11)

**Parametric knowledge** — Facts encoded in LLM weights during pre-training (Ch 4)

**Preference alignment** — Training on chosen vs. rejected responses for subjective quality (Ch 6)

**QLoRA** — Quantized LoRA for memory-efficient fine-tuning (Ch 5)

**Query expansion** — LLM-generated query variants to improve RAG recall (Ch 9)

**RAG (Retrieval-Augmented Generation)** — Retrieve external data, augment prompt, generate (Ch 4)

**Rejected answer** — Undesired response in a DPO preference pair (Ch 6)

**Reranking** — Rescoring retrieved chunks for precision before prompt injection (Ch 9)

**Self-querying** — Extracting metadata filters from natural language queries for vector search (Ch 9)

**SFT (Supervised Fine-Tuning)** — Fine-tuning on instruction–answer pairs (Ch 5)

**Speculative decoding** — Draft model predicts tokens; target model validates prefix (Ch 8)

**Static KV cache** — Pre-allocated KV cache enabling torch.compile optimization (Ch 8)

**Training pipeline** — FTI component: features → trained model → model registry (Ch 1, Ch 5–6)

**Training-serving skew** — Features computed differently at train vs. inference time (Ch 1, Ch 4)

**Vanilla RAG** — Basic three-module RAG: ingestion, retrieval, generation (Ch 4)

**Vector DB** — Database indexing embeddings for similarity search (Qdrant in LLM Twin) (Ch 2, Ch 4)

**ZenML** — MLOps orchestrator for pipelines, artifacts, and metadata (Ch 2–3, Ch 11)
