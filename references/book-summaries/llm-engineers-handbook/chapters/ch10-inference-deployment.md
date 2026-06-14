# Chapter 10: Inference Pipeline Deployment

## Core Idea
Choose deployment architecture by balancing throughput, latency, data shape, and infrastructure cost — the LLM Twin uses online real-time inference (FastAPI + SageMaker) with autoscaling, while understanding when async or batch transform fits better.

## Frameworks Introduced
- **Four deployment criteria**: Throughput (RPS), latency (ms per request), data (format/volume), infrastructure (GPU/CPU/network).
  - When to use: Before any model serving decision.
  - How: Map product UX requirements → pick deployment type → pick monolith vs. microservices.
- **Three inference deployment types**: Online real-time, asynchronous (queued), offline batch transform.
  - When to use: Real-time = chatbots; async = long jobs (summarize 100 docs); batch = overnight predictions OK.
  - How: REST/gRPC for real-time; queue + polling/push for async; scheduled batch jobs for offline.
- **Monolithic vs. microservices serving**: Single service with model + preprocessing vs. split retriever, ranker, generator services.
  - When to use: Monolith for PoC/small teams; microservices when independent scaling of retrieval vs. LLM needed.

## Key Concepts
- **Online real-time inference**: Synchronous HTTP — client waits; WebSocket token streaming for chat UX.
- **Asynchronous inference**: Queue requests, process at controlled parallelism — handles traffic spikes cheaply.
- **Offline batch transform**: Schedule predictions on large datasets — highest throughput, highest latency.
- **Latency–throughput trade-off**: Batching raises throughput but can raise per-request latency.
- **FastAPI**: Central entry point for LLM Twin users in book's deployment tutorial.
- **SageMaker inference endpoint**: Hosts quantized TwinLlama with autoscaling policies.
- **Autoscaling**: Scale on CPU, request count, queue depth, or combinations.

## Mental Models
- 53% of mobile users abandon sites loading >3 seconds — latency is a product killer even with a great model.
- Async queues absorb spikes without 10× VM costs — pay with higher latency.
- gRPC for internal ML services (speed); REST for public APIs (accessibility).

## Anti-patterns
- **Batch deployment for chat UX**: Users won't wait hours for LinkedIn post generation.
- **No autoscaling on real-time LLM**: Traffic spikes cause timeouts and churn.
- **Ignoring serialization overhead**: Latency = network I/O + (de)serialization + model inference — profile all three.

## Worked Example
**LLM Twin deployment stack**:
1. Custom fine-tuned LLM → SageMaker endpoint (quantized, Ch 8 optimizations)
2. FastAPI server: accepts user prompts, calls RAG retrieval module, forwards augmented prompt to SageMaker
3. Online real-time with token streaming for responsive UX
4. Autoscaling on request rate for SageMaker endpoint
5. Model registry (Hugging Face) versions which checkpoint is deployed

## Key Takeaways
1. Ask throughput, latency, concurrency, scaling, cost, and data-type questions before choosing deployment mode.
2. Online real-time for interactive LLM Twin; async for long-running batch content jobs.
3. FastAPI + SageMaker is the book's production pattern for the inference pipeline.
4. Monolithic serving is fine to start; split when retrieval and generation need different scale profiles.
5. Autoscaling prevents crash-under-spike while controlling GPU costs during quiet periods.

## Connects To
- **Ch 8**: Quantization and inference engines inform SageMaker endpoint config.
- **Ch 9**: RAG retrieval module sits in front of SageMaker LLM call.
- **Ch 11**: CI/CD automates deployment; monitoring tracks production prompts.
