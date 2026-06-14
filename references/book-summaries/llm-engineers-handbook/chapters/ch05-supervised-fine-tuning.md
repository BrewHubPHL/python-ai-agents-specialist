# Chapter 5: Supervised Fine-Tuning

## Core Idea
SFT bridges general pre-trained LLMs to task-specific conversational agents by training on instruction–answer pairs; dataset quality (accuracy, diversity, complexity) matters more than raw volume, and parameter-efficient methods (LoRA, QLoRA) make fine-tuning accessible on consumer hardware.

## Frameworks Introduced
- **Instruction dataset framework**: Pairs of instruction + answer (optional system/input fields per Alpaca template).
  - When to use: Any SFT beyond base model chat formatting.
  - How: Curate representative samples → filter (rules, dedup, decontamination) → evaluate quality → train.
- **Data quality triad**: Accuracy, diversity, complexity — filter ruthlessly on all three.
  - When to use: Every instruction dataset before training.
  - How: Rule-based filters (length, keywords, format) + semantic dedup + benchmark decontamination.
- **SFT techniques**: Full fine-tuning, LoRA (Low-Rank Adaptation), QLoRA (quantized LoRA).
  - When to use: LoRA/QLoRA when GPU memory is limited; full FT when maximum quality needed.
  - How: LoRA injects low-rank matrices into attention layers; QLoRA quantizes base weights to 4-bit.

## Key Concepts
- **Alpaca template**: system + instruction + input + output fields for chat formatting.
- **Task-specific vs. domain-specific fine-tuning**: Task = one function (summarize); domain = field vocabulary (medicine, law).
- **Data quantity heuristics**: ~1M samples for general-purpose instruct; 100–100K for task-specific; LIMA showed 1K high-quality samples can work for 70B models.
- **Rule-based filtering**: Length thresholds, keyword exclusion, format checking.
- **Data deduplication**: Prevents overfitting and biased performance on repeated samples.
- **Data decontamination**: Remove samples overlapping benchmark test sets (e.g., MMLU).
- **TwinLlama-3.1-8B**: Book's SFT output model on Hugging Face.

## Mental Models
- Creating the instruction dataset is usually harder than running training — budget time for curation, not just GPU hours.
- Few-shot prompting is an alternative to task-specific SFT when you have a large base model and limited data.
- Smaller models (7B) need more samples than 70B models to learn chat templates — size and quality interact.

## Anti-patterns
- **Volume without quality**: 1M noisy samples underperform 10K curated ones.
- **Benchmark contamination**: +10 MMLU points after SFT likely means test leakage, not real improvement.
- **Training on instructions only**: For chat models, often train on answers only to preserve instruction understanding.

## Worked Example
**SlimOrca-style sample structure**:
| Field | Content |
|---|---|
| System | "You are a helpful assistant... Think like you are answering to a five year old." |
| Instruction | "Concepts: building, shop, town — Write a sentence that includes all these words." |
| Output | "In our little town, there is a shop inside a big building where people go to buy their favorite toys and candies." |

Quality dimensions: factually accurate, diverse topics/styles, non-trivial reasoning.

## Key Takeaways
1. SFT teaches chat format and domain/task specialization on top of pre-training.
2. Curate for accuracy, diversity, and complexity — then filter, deduplicate, and decontaminate.
3. General-purpose instruct models need ~1M samples; task/domain models need far fewer.
4. LoRA/QLoRA enable fine-tuning 8B models on limited GPU memory.
5. Transform Ch 3 scraped data into instruction pairs for the LLM Twin writing use case.

## Connects To
- **Ch 3**: Raw crawled text becomes instruction dataset source material.
- **Ch 6**: DPO preference alignment refines SFT output for authentic writing style.
- **Ch 7**: Evaluate fine-tuned model vs. base on instruction-following benchmarks.
