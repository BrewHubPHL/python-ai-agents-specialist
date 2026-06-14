# Patterns — LLM Engineer's Handbook

## FTI Pipeline Decomposition
**When to use**: Designing any production ML or LLM system beyond a notebook.
**How**: Split into feature pipeline (→ feature store), training pipeline (→ model registry), inference pipeline (reads both). Keep interfaces stable even as internal complexity grows.
**Trade-offs**: More moving parts than monolith, but enables independent scaling, team ownership, and technology choices per pipeline.

## ETL with Crawler Dispatcher
**When to use**: Collecting multi-platform personal/content data for LLM training.
**How**: Input user + URLs → detect domain → dispatch to specialized crawler → normalize to document categories (article/repo/post) → load MongoDB.
**Trade-offs**: Login-based crawlers (Medium, LinkedIn) are fragile; category abstraction avoids per-platform schema explosion.

## Vanilla RAG Three-Module Split
**When to use**: Any GenAI app needing private or fresh knowledge without constant fine-tuning.
**How**: Ingestion (batch: chunk, embed, load vector DB) + Retrieval (embed query, top-K similarity) + Generation (template + context + LLM).
**Trade-offs**: Requires identical preprocessing at ingest and query; naive single-query retrieval may miss relevant docs.

## Advanced RAG Retrieval Stack
**When to use**: Production RAG where recall and precision both matter.
**How**: Query expansion (N variants) → self-querying (metadata filters) → filtered vector search → aggregate N×K → cross-encoder rerank → top-K → prompt → LLM.
**Trade-offs**: More LLM API calls at retrieval time; significantly better context quality.

## Instruction Dataset Curation Pipeline
**When to use**: Supervised fine-tuning for any custom LLM behavior.
**How**: Curate instruction–answer pairs → rule-based filter (length, keywords, format) → deduplicate → decontaminate against benchmarks → quality evaluation on accuracy/diversity/complexity.
**Trade-offs**: Labor-intensive; quality matters more than raw count.

## DPO Preference Alignment
**When to use**: Subjective output quality (tone, naturalness) that SFT cannot capture.
**How**: Build instruction + chosen + rejected triples → run DPO on SFT checkpoint. Generate prefs via LLM diversity; evaluate with humans or stronger LLM.
**Trade-offs**: Fewer samples needed than SFT for style tasks; wrong rejected pairs teach bad behavior.

## Parameter-Efficient Fine-Tuning (LoRA/QLoRA)
**When to use**: Fine-tuning 7B–8B models on limited GPU memory.
**How**: Inject low-rank adapters into attention layers; QLoRA quantizes base weights to 4-bit during training.
**Trade-offs**: Slightly lower max quality vs. full fine-tune; dramatically lower memory and cost.

## Inference Optimization Stack
**When to use**: Serving decoder-only LLMs with latency/throughput requirements.
**How**: KV cache → static cache + torch.compile → continuous batching → speculative decoding → quantization → TGI/vLLM/TensorRT-LLM.
**Trade-offs**: Speculative decoding needs matched tokenizers; quantization may slightly reduce quality.

## Deployment Type Selection
**When to use**: Choosing how to serve model predictions.
**How**: Map UX latency needs → online real-time (chat), async queue (long jobs), or batch transform (overnight). Balance throughput vs. latency with batching strategy.
**Trade-offs**: Real-time costs more GPU idle time; async adds latency; batch unsuitable for interactive UX.

## MLOps/LLMOps Operationalization
**When to use**: Moving from PoC to production LLM application.
**How**: CI/CD (GitHub Actions) + CT (ZenML retrain triggers) + monitoring (Opik prompts) + version code/data/models/prompts independently.
**Trade-offs**: Infrastructure complexity; essential for reproducibility and drift response.
