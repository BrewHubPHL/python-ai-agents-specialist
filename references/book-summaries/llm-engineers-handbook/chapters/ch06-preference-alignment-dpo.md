# Chapter 6: Fine-Tuning with Preference Alignment

## Core Idea
SFT alone cannot capture nuanced human preferences; **Direct Preference Optimization (DPO)** trains models on chosen vs. rejected response pairs without a separate reward model, making writing style more authentic with relatively few preference samples.

## Frameworks Introduced
- **Preference alignment**: Incorporate human or AI feedback beyond supervised instruction pairs.
  - When to use: Subjective quality (tone, naturalness, engagement) that SFT misses.
  - How: Build preference dataset → run DPO (or RLHF/ORPO variants) on SFT checkpoint.
- **DPO (Direct Preference Optimization)**: Each sample = instruction + chosen answer + rejected answer.
  - When to use: Style alignment, content moderation nuance, summarization quality, creative writing tone.
  - How: Model learns to increase probability of chosen, decrease rejected — no reward model training loop.
- **Preference data generation matrix**: Human/LLM × generate/evaluate — four methods with cost/quality trade-offs.
  - When to use: Scaling preference data beyond manual labeling.
  - How: Prefer LLM-generated + human-evaluated or high-quality vs. low-quality model pairs.

## Key Concepts
- **Chosen vs. rejected response**: Rejected defines behavior to eliminate — not optional decoration.
- **DPO sample efficiency**: 100–10K pairs for task-specific alignment; millions for general-purpose provider alignment.
- **Less destructive than SFT**: DPO has milder impact — useful for healing models after merge/prune.
- **Temperature for synthetic prefs**: Low temp for code; higher for creative writing diversity.
- **TwinLlama-3.1-8B-DPO**: Preference-aligned model artifact on Hugging Face.
- **Intel/orca_dpo_pairs**: High-quality model outputs vs. low-quality model outputs as chosen/rejected.

## Mental Models
- Preference data captures *why* one answer is better, not just *what* the right answer is — essential for subjective tasks.
- LLM-generated + human-evaluated balances scale and quality — humans judge better than they write from scratch.
- Compare your fine-tuned output to human-written text to copy authentic tone (LLM Twin writing style goal).

## Anti-patterns
- **SFT-only for style goals**: SFT teaches format; DPO teaches preference — use both for authentic voice.
- **Missing rejected pairs**: Without rejected examples, preference data collapses to plain instruction tuning.
- **Multi-turn preference data**: Major libraries extract only first/last turn — design single-turn prefs for DPO tooling compatibility.

## Worked Example
**DPO sample** (instruction → chosen vs. rejected):
| Field | Content |
|---|---|
| Instruction | "Tell me a joke about octopuses." |
| Chosen | "Why don't octopuses play cards in casinos? Because they can't count past eight." |
| Rejected | "How many tickles does it take to make an octopus laugh? Ten tickles." |

For LLM Twin: chosen = human-like writing; rejected = generic ChatGPT-style wordiness.

## Key Takeaways
1. Preference alignment fixes SFT's blind spot for subjective, nuanced output quality.
2. DPO format is simple: instruction + preferred + rejected response per row.
3. Task-specific alignment (writing style) needs 100–10K pairs; quality beats quantity.
4. Generate prefs with LLM diversity (multi-model, temperature variation) then evaluate.
5. Run DPO on TwinLlama SFT checkpoint to reduce artificial tone.

## Connects To
- **Ch 5**: DPO follows SFT on same base checkpoint.
- **Ch 7**: Evaluate aligned model on conversational and instruction benchmarks.
- **DPO paper**: Direct Preference Optimization (Rafailov et al.) — cited in references.
