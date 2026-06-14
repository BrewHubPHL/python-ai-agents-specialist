# Chapter 11: MLOps and LLMOps

## Core Idea
LLMOps extends MLOps (which extends DevOps) with LLM-specific concerns — prompt versioning/monitoring, guardrails, and feedback loops — while the LLM Twin adds CI/CD (GitHub Actions), continuous training (ZenML), and prompt monitoring (Opik) on top of cloud-deployed pipelines.

## Frameworks Introduced
- **DevOps → MLOps → LLMOps stack**: Each layer adds first-class citizens — code → code+data+model → code+data+model+prompts.
  - When to use: Structuring any production LLM team's practices.
  - How: Inherit automation, versioning, CI/CD; add experiment tracking, feature stores, model registries; add prompt ops.
- **Six MLOps principles**: Automation, versioning, experiment tracking, testing, monitoring, reproducibility.
  - When to use: Every production ML system design review.
  - How: CT pipelines retrain on triggers; Git for code; registries for models; DVC/artifacts for data.
- **LLMOps-specific concerns**: Prompt monitoring/versioning, input/output guardrails, feedback loops for fine-tuning data, cost scaling at trillion-token scale.
  - When to use: Any deployed LLM application beyond a demo.
  - How: Opik for prompt traces; CI/CD for pipeline code; CT for scheduled retraining; guardrails for toxicity/PII.

## Key Concepts
- **CI/CD**: GitHub Actions builds, tests, deploys on merge to staging/production.
- **CT (Continuous Training)**: Automated retrain when new data, performance drop, or distribution shift detected.
- **Model registry / feature store / orchestrator**: MLOps core components used throughout book (ZenML, Hugging Face, MongoDB/Qdrant as logical stores).
- **Deployment environments**: dev → staging → production with promotion gates.
- **DS vs. MLE vs. MLOps**: Researcher builds model → MLE makes it modular/API-accessible → MLOps puts it on infrastructure with automation.
- **ZenML AWS deployment**: All pipelines (not just inference) deployed to cloud in this chapter.

## Mental Models
- In DevOps, code changes trigger builds; in MLOps, *data* changes can trigger retrains without any code change.
- Most companies adopt foundation models (Llama) — LLMOps focus is prompt ops and RAG, not training GPT-scale clusters.
- Small teams wear all three hats (DS/MLE/MLOps) — role separation is a big-company luxury.

## Anti-patterns
- **Manual pipeline runs in production**: Defeats reproducibility and delays response to drift.
- **Unversioned prompts**: Can't explain which prompt version generated a bad production answer.
- **Monitoring only infrastructure, not prompts**: GPU uptime tells you nothing about hallucination rate or user satisfaction.

## Worked Example
**LLMOps additions to LLM Twin**:
1. **CI/CD** (GitHub Actions): lint + test on PR; auto-deploy pipelines to staging on merge
2. **CT** (ZenML): trigger fine-tuning pipeline when new crawled data exceeds threshold
3. **Monitoring** (Opik): log every prompt + retrieved context + generated answer; alert on quality regressions
4. **Cloud deploy**: ZenML pipelines on AWS (ETL, feature, training, inference) — not just SageMaker endpoint

## Key Takeaways
1. LLMOps = MLOps + prompt versioning, guardrails, feedback loops, and LLM-scale cost concerns.
2. MLOps makes data and models first-class citizens alongside code in CI/CD.
3. Six principles: automate, version, track experiments, test (code+data+model), monitor, reproduce.
4. Implement CI/CD + CT + monitoring as the natural final step after building the application.
5. Deploy all ZenML pipelines to AWS — production is pipelines + endpoints, not endpoints alone.

## Connects To
- **Ch 1**: FTI architecture is the structural backbone MLOps operationalizes.
- **Ch 2**: ZenML, Opik, Comet introduced as tools; Ch 11 connects theory.
- **Ch 10**: Inference deployment is one pipeline under the broader MLOps umbrella.
