---
title: "Serving Gemma 3 12B at Scale"
date: 2025-02-15
draft: true
summary: "Running 11 NLP packages on vLLM for 10K posts/day — batching, memory management, and latency tradeoffs."
tags:
  - "LLM Inference"
  - "vLLM"
  - "Production ML"
---

At Saptang Labs, we serve 11 transformer-based NLP packages through a single Gemma 3 12B instance running on vLLM. The system processes approximately 10,000 posts per day.

## The architecture

Each NLP package (entity extraction, sentiment analysis, topic classification, etc.) is defined as a structured prompt template. All packages share the same model instance — no fine-tuning, just carefully engineered prompts with output schemas.

## Batching strategy

vLLM's continuous batching handles most of the throughput optimization automatically. The key decisions were:

1. **Max batch size**: Tuned to GPU memory constraints (A100 80GB)
2. **Prefix caching**: Shared system prompts across packages reduce KV-cache overhead
3. **Priority queuing**: Time-sensitive packages get higher scheduling priority

## Memory management

With 12B parameters in FP16 plus KV-cache for concurrent requests, memory budgeting is critical. We settled on a 70/30 split — 70% for model weights and static allocation, 30% for dynamic KV-cache.

## Latency tradeoffs

Batching improves throughput but adds latency for individual requests. For our use case, p95 latency under 2 seconds was acceptable since the downstream pipeline is asynchronous.

*Detailed benchmarks and configuration specifics coming soon.*
