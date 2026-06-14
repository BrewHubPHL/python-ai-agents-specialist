# Chapter 4: RAG Feature Pipeline

## Core Idea
RAG injects external/private data into LLM prompts without constant fine-tuning; the feature pipeline implements ingestion (chunk → embed → load to Qdrant) while vanilla RAG's three modules — ingestion, retrieval, generation — must share preprocessing to avoid training-serving skew.

## Frameworks Introduced
- **Vanilla RAG framework**: Three independent modules — ingestion pipeline, retrieval pipeline, generation pipeline.
  - When to use: Any GenAI app needing private, fresh, or domain-specific knowledge.
  - How: Backend ingestion populates vector DB on schedule; client query → retrieve top-K → augment prompt → LLM generate.
- **RAG value proposition**: Solves hallucinations (context as single source of truth) and stale/private data (inject without retraining).
  - When to use: Always when LLM must cite your data, not parametric knowledge alone.
- **Advanced RAG** (preview): Query expansion, self-querying, reranking — detailed in Ch 9 at inference time.

## Key Concepts
- **Parametric knowledge**: What the LLM learned during pre-training — bounded by cutoff date.
- **Ingestion pipeline stages**: Extract → clean → chunk → embed → load (with metadata: source URL, publish date).
- **Retrieval pipeline**: Embed user query → cosine similarity → top-K chunks augment prompt.
- **Cosine distance**: Default similarity metric in embedding space; must match ingestion preprocessing.
- **Training-serving skew in RAG**: User query must be cleaned/chunked/embedded identically to ingestion path.
- **Embeddings**: Dense vectors capturing semantic similarity; enable vector DB search.
- **Chunking**: Split documents at semantic boundaries so retrieval injects only relevant sections.
- **OVM (Object-Vector Mapping)**: Domain pattern mapping documents to vector DB entries (used for Query entity in Ch 9).

## Mental Models
- RAG makes the LLM a *reasoning engine*; retrieved context is the *source of truth* — evaluate answers against context, not model memory.
- Think ingestion as offline batch/streaming; retrieval+generation as online on-demand — same embedding model bridges both.
- Chunk quality determines retrieval quality — bad chunks mean bad prompts regardless of LLM size.

## Anti-patterns
- **Skipping context-only answers**: Prompts without "answer only from context" allow hallucination even with RAG.
- **Mismatched embedding spaces**: Different embed models or preprocessing at ingest vs. query makes distance meaningless.
- **Giant chunks**: Dumping whole documents into prompts wastes tokens and dilutes relevance.

## Code Examples
```python
system_template = """You are a helpful assistant who answers all the user's questions politely."""
prompt_template = """
Answer the user's question using only the provided context. If you cannot
answer using the context, respond with "I don't know."
Context: {context}
User question: {user_question}
"""
retrieved_context = retrieve(user_question)
prompt = f"{system_template}\n"
prompt += prompt_template.format(context=retrieved_context, user_question=user_question)
answer = llm(prompt)
```
- **What it demonstrates**: Context-grounded generation template with explicit refusal behavior.

## Reference Tables
| RAG module | Responsibility | Runs when |
|---|---|---|
| Ingestion | Extract, clean, chunk, embed, load | Scheduled/batch |
| Retrieval | Embed query, similarity search | Per user request |
| Generation | Template + context + LLM | Per user request |

## Worked Example
**Vanilla RAG request flow**:
1. Ingestion pipeline (scheduled) populates Qdrant with chunked embeddings + metadata
2. User asks: "Write an article about X"
3. Retrieval embeds query, finds top-K similar chunks via cosine distance
4. Generation builds prompt from system template + context + question
5. LLM returns answer grounded in retrieved chunks

## Key Takeaways
1. RAG avoids costly retraining for new/private data and reduces hallucinations by grounding answers.
2. Ingestion, retrieval, and generation are separate services — design interfaces, not monoliths.
3. Embeddings must use identical preprocessing at ingest and query time.
4. Chunking strategy is as important as embedding model choice.
5. Version prompt templates with MLOps practices (Git, DB, or LangFuse/Opik).

## Connects To
- **Ch 3**: Raw MongoDB documents feed this feature pipeline.
- **Ch 9**: Advanced retrieval (query expansion, reranking) at inference.
- **Ch 7**: RAG system evaluation extends beyond model-only benchmarks.
