# Chapter 2: Tooling and Installation

## Core Idea
The LLM Twin stack is Python 3.11 + Poetry + Poe the Poet for local dev, Docker-hosted MLOps/LLMOps services (ZenML, MongoDB, Qdrant), Hugging Face as model registry, and AWS SageMaker for cloud training/inference.

## Frameworks Introduced
- **Python project trinity**: pyenv (interpreter) + Poetry (deps + venv) + Poe the Poet (task runner).
  - When to use: Any reproducible Python ML project.
  - How: `pyenv local 3.11.8` → `poetry install` → `poetry poe <task>`.
- **ZenML orchestration**: Pipelines, steps, artifacts, metadata — bridges ML code and MLOps.
  - When to use: Multi-pipeline systems (ETL, feature, training, inference).
  - How: `@pipeline` / `@step` decorators; configure via YAML; run locally or on AWS.
- **Local infrastructure stack**: `poetry poe local-infrastructure-up` spins ZenML + MongoDB + Qdrant via Docker.

## Key Concepts
- **Poetry lock file**: Pins full dependency tree for "works on my machine" elimination.
- **Hugging Face model registry**: Centralized model versions (TwinLlama-3.1-8B, TwinLlama-3.1-8B-DPO).
- **ZenML artifacts/metadata**: Every step output versioned with query params and retrieval info.
- **Comet ML**: Experiment tracking for training runs.
- **Opik**: Prompt monitoring for LLMOps.
- **MongoDB**: NoSQL store for unstructured crawled text (data warehouse).
- **Qdrant**: Vector DB for embeddings.
- **AWS SageMaker**: Managed GPU training and inference endpoints.
- **uv**: Rust-based Poetry alternative (faster, worth evaluating).

## Mental Models
- Use Poe the Poet as living CLI documentation — tasks in `pyproject.toml` replace README command lists.
- Run MLOps theory last (Ch 11) but tools first (Ch 2) — hands-on before abstractions.
- Docker-compose local stack mirrors cloud topology without AWS costs during development.

## Anti-patterns
- **Global Python dependencies**: Installing numpy globally breaks cross-project version isolation.
- **README-only runbooks**: Undocumented shell commands don't survive team churn — encode in Poe tasks.
- **Skipping `.env` credentials**: OpenAI and Hugging Face tokens required for full pipeline runs.

## Code Examples
```bash
git clone https://github.com/PacktPublishing/LLM-Engineers-Handbook.git
cd LLM-Engineers-Handbook
poetry install --without aws
poetry self add 'poethepoet[poetry_plugin]'
poetry poe local-infrastructure-up
```
- **What it demonstrates**: Standard project bootstrap and local MLOps infrastructure.

```toml
[tool.poe.tasks]
test = "pytest"
format = "black ."
start = "python main.py"
```
- **What it demonstrates**: Task aliases executed via `poetry poe test`.

## Worked Example
**Local dev checklist**:
1. Install pyenv → Python 3.11.8
2. Clone LLM-Engineers-Handbook repo
3. `poetry install --without aws`
4. Fill `.env` (OpenAI, Hugging Face, AWS keys per README)
5. Docker 27.1.1+ → `poetry poe local-infrastructure-up`
6. Access ZenML dashboard at `http://127.0.0.1:8237/`

## Key Takeaways
1. Pin Python 3.11.8 and lock dependencies before touching ML code.
2. ZenML is the orchestration backbone for every LLM Twin pipeline.
3. Hugging Face hosts fine-tuned model artifacts; Comet tracks experiments; Opik monitors prompts.
4. MongoDB + Qdrant split raw text warehouse from vector search index.
5. SageMaker handles GPU-heavy training/inference when local hardware is insufficient.

## Connects To
- **Ch 3**: `digital_data_etl` ZenML pipeline introduced here, implemented next chapter.
- **Ch 11**: Full cloud deployment of ZenML pipelines to AWS.
- **ZenML**: Orchestrator pattern from MLOps stack.
