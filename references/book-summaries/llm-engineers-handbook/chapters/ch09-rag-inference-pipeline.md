# Chapter 9: RAG Inference Pipeline

## Core Idea
Most RAG engineering lives in the retrieval module — implement advanced pre-retrieval (query expansion, self-querying), filtered vector search, and post-retrieval reranking before augmenting the prompt and calling the SageMaker-hosted LLM Twin.

## Frameworks Introduced
- **Advanced RAG pipeline flow**: User query → query expansion → self-querying → filtered vector search (×N queries) → aggregate N×K results → rerank to top-K → build prompt → LLM → answer.
  - When to use: Production RAG where naive single-vector search misses relevant context.
  - How: Separate retrieval service from generation; feature pipeline and retrieval run on independent schedules.
- **Query expansion**: LLM generates N alternative queries capturing different facets of the original question.
  - When to use: Narrow or vague user queries that under-cover embedding space.
  - How: Zero-shot prompt to GPT-4o-mini (configurable via `OPENAI_MODEL_ID`) → embed each variant separately.
- **Self-querying**: Extract metadata filters (e.g., `author_id`) from natural language query before vector search.
  - When to use: Multi-author corpora where results must be scoped to a specific person's content.
  - How: LLM parses query → populate `Query` entity filters → Qdrant filtered search.
- **Reranking**: Cross-encoder scores N×K candidates against original query; keep top-K by relevance score [0,1].
  - When to use: After multi-query retrieval produces noisy supersets.
  - How: Neural cross-encoder rescores chunks relative to initial user query (not expanded variants).

## Key Concepts
- **RAGStep interface**: Abstract base with `generate(query)` and `mock` flag for cost-free dev.
- **Query / EmbeddedQuery domain entities**: `content`, `author_id`, `author_full_name`, `metadata`, `embedding`.
- **PromptTemplateFactory**: Standardizes LangChain `PromptTemplate` creation.
- **Filtered vector search**: Metadata filters shrink search space before cosine similarity.
- **Feature freshness pattern**: Feature pipeline writes Qdrant on schedule; retrieval reads latest on each request.

## Mental Models
- Don't over-engineer a separate "prompt augmentation module" — prompt building belongs in the inference service.
- Single-query vector search covers one point in embedding space; expansion explores semantically related regions.
- Pre-retrieval optimizes *recall*; reranking optimizes *precision* — you need both.

## Anti-patterns
- **Monolithic RAG in one function**: Retrieval, prompt build, and LLM call should be separable testable modules.
- **Skipping reranking after query expansion**: N×K results without rescoring injects irrelevant context.
- **Heavy LangChain dependency**: Use LangChain objects (PromptTemplate) sparingly — understand the engineering underneath.

## Code Examples
```python
class RAGStep(ABC):
    def __init__(self, mock: bool = False) -> None:
        self._mock = mock

    @abstractmethod
    def generate(self, query: Query, *args, **kwargs) -> Any:
        pass

class Query(VectorBaseDocument):
    content: str
    author_id: UUID4 | None = None
    author_full_name: str | None = None
    metadata: dict = Field(default_factory=dict)
```
- **What it demonstrates**: Standardized interfaces for advanced RAG steps and query metadata.

## Worked Example
**LinkedIn post generation flow**:
1. User: "Write a post about fine-tuning LLMs" (scoped to their author_id via self-querying)
2. Query expansion → 3 variant queries about fine-tuning, LoRA, dataset curation
3. Each variant: filtered vector search top-5 chunks from user's past posts/articles
4. Aggregate 15 chunks → cross-encoder rerank → top-5 most relevant
5. Prompt template injects context + user query → TwinLlama on SageMaker endpoint
6. Return generated post in user's voice

## Key Takeaways
1. Retrieval is where advanced RAG logic lives — not at the LLM API call.
2. Query expansion improves recall; reranking improves precision.
3. Self-querying enables author-scoped search for personalized twins.
4. Feature pipeline (batch) and retrieval module (on-demand) stay decoupled via Qdrant.
5. Use `mock=True` on RAGSteps during development to avoid OpenAI API costs.

## Connects To
- **Ch 4**: Vanilla RAG theory; Ch 9 implements production retrieval.
- **Ch 10**: SageMaker hosts the LLM; this chapter builds the RAG wrapper.
- **Ch 8**: Optimized inference on the SageMaker endpoint.
