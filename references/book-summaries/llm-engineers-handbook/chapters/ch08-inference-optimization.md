# Chapter 8: Inference Optimization

## Core Idea
LLM inference bottlenecks are sequential token generation and memory pressure; combine KV cache (static + torch.compile), continuous batching, speculative decoding, model parallelism, and weight quantization to achieve 2–4× speedups without quality loss.

## Frameworks Introduced
- **KV cache**: Store attention key-value pairs so each new token doesn't recompute full context.
  - When to use: Every decoder-only inference deployment.
  - How: Cache grows per token; static pre-allocation enables `torch.compile` for up to 4× forward-pass speedup.
- **Continuous batching (in-flight batching)**: Replace completed requests immediately — no waiting for batch stragglers.
  - When to use: Variable prompt/output lengths at serving time.
  - How: Evict finished sequences, insert new requests; balance prefill vs. generation ratio.
- **Speculative decoding**: Small draft model predicts N tokens; large model validates longest matching prefix.
  - When to use: GPU has spare capacity during sequential generation.
  - How: `model.generate(assistant_model=draft_model)` — same tokenizer required; ~90% match → 3–4× speedup.

## Key Concepts
- **Decoder-only inference loop**: Tokenize → embed → compute KV for prompt → autoregressive token generation (sequential bottleneck).
- **Static KV cache**: Pre-allocate max cache size for compilation compatibility.
- **Prompt lookup decoding**: Speculative variant using n-gram overlap between prompt and output (summarization).
- **Medusa**: Joint fine-tune speculation heads on main model for higher draft fidelity.
- **Model parallelism**: Split layers across GPUs when model exceeds single-device memory.
- **Quantization**: INT8/INT4 weights reduce memory and increase throughput (GPTQ, AWQ, etc.).
- **Inference engines**: TGI (Text Generation Inference), vLLM, TensorRT-LLM — implement batching + KV optimizations natively.

## Mental Models
- Steps 1–2 (prompt encoding) parallelize well; Step 3 (token-by-token) is the bottleneck — optimize there.
- KV cache memory scales with sequence length × layers × heads × precision — monitor before raising batch size.
- Speculative decoding trades draft model quality for throughput — draft must share tokenizer with target model.

## Anti-patterns
- **Naive single-request serving**: No batching leaves GPU underutilized.
- **Mismatched tokenizers in speculative decoding**: Draft tokens won't align with target model validation.
- **Ignoring KV cache memory**: Long contexts can exceed 2GB KV alone on 7B models — caps batch size.

## Code Examples
```python
model.generation_config.cache_implementation = "static"
compiled_model = torch.compile(model, mode="reduce-overhead", fullgraph=True)

outputs = model.generate(
    **inputs,
    do_sample=True,
    assistant_model=draft_model,
    temperature=0.7,
    max_new_tokens=64,
)
```
- **What it demonstrates**: Static KV cache + speculative decoding via Hugging Face transformers.

## Reference Tables
| Technique | Target | Typical gain |
|---|---|---|
| KV cache | Avoid recomputing attention | Foundational |
| Static KV + compile | Forward pass | Up to 4× |
| Continuous batching | GPU utilization | Higher throughput |
| Speculative decoding | Token generation | 2–4× (draft quality dependent) |
| Quantization | Memory + speed | Model-size dependent |

## Worked Example
**Optimization stack for serving**:
1. Enable static KV cache + `torch.compile`
2. Deploy on vLLM/TGI with continuous batching
3. Add Qwen-0.5B draft model for Qwen-1.8B target (speculative decoding)
4. Quantize weights to 4-bit if memory-bound
5. Use tensor parallelism if model spans multiple GPUs

## Key Takeaways
1. Sequential token generation is the fundamental latency bottleneck in decoder-only models.
2. KV cache is mandatory; static cache unlocks compilation optimizations.
3. Continuous batching maximizes throughput for variable-length requests.
4. Speculative decoding uses a small draft model to predict multiple tokens per large-model step.
5. Choose TGI, vLLM, or TensorRT-LLM for production — don't roll your own batching.

## Connects To
- **Ch 10**: Deploy optimized model to SageMaker endpoints.
- **Ch 9**: RAG inference latency includes retrieval + LLM — optimize both.
- **Quantization**: Pairs with QLoRA training patterns from Ch 5.
