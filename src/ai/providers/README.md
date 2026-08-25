# Providers <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

A comparison of reputable LLM API providers, with a short background on each and its standout
capabilities.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
- [OpenRouter](#openrouter)
- [Together AI](#together-ai)
- [Fireworks AI](#fireworks-ai)
- [Groq](#groq)
- [DeepInfra](#deepinfra)
- [SiliconFlow](#siliconflow)

## Overview
Inference is not a commodity: a provider that saves on token cost can end up costing more in
engineering time if models get deprecated without warning, rate limits throttle an agent mid-run,
or a fine-tuning workflow requires switching vendors entirely. The right choice depends on whether
you value reach (many models behind one key), raw speed, price, or fine-tuning/custom deployment
support.

## OpenRouter
OpenRouter is an aggregation marketplace rather than a first-party inference host: one API key
gives access to models from dozens of underlying providers (including many of the ones below),
with automatic routing and fallback between them.

* Broadest model reach of any provider — one key, hundreds of models across every major lab
* Automatic fallback/routing if a given upstream provider is degraded or rate-limited
* `openrouter/free` router auto-selects from free-tier models, useful for prototyping without cost
* OpenAI-compatible API, making it a drop-in `baseURL` swap for most existing tooling (incl. OpenCode)
* Ideal when you want maximum model choice and don't need multimodal support, kernel-level
  optimization, or dedicated/custom deployments

## Together AI
Together AI positions itself around model breadth plus first-party fine-tuning and training
infrastructure, rather than just serving inference.

* Wide catalog of open-weight models, including many Chinese labs (DeepSeek, Qwen, etc.)
* Strong fine-tuning and custom training support, not just inference
* Competitive pricing on open models
* Good fit if you expect to fine-tune or customize a model rather than just call it as-is

## Fireworks AI
Fireworks AI focuses on raw inference speed and price for open-weight models, with its own
optimized serving stack.

* Leads (alongside DeepInfra) on raw open-model price and speed
* Custom-optimized inference stack (FireAttention) for lower latency on popular open models
* Supports fine-tuning and custom model deployment
* Good default when cost and throughput matter more than provider breadth

## Groq
Groq is not a model lab — it's a hardware/inference company built around its own LPU
(Language Processing Unit) chips, purpose-built for LLM token generation rather than general GPU compute.

* Consistently benchmarks as the fastest inference provider available, by a wide margin
* Purpose-built LPU hardware rather than repurposed GPUs
* Best fit when latency/tokens-per-second is the primary constraint (e.g. voice agents, live UX)
* Narrower model catalog than aggregators like OpenRouter — optimized for a curated set of
  popular open models rather than everything available

## DeepInfra
DeepInfra is a low-cost, high-throughput inference host focused squarely on serving open-weight
models cheaply at scale.

* Leads (alongside Fireworks) on raw open-model price and speed
* Simple, predictable pay-per-token pricing across a broad set of open models
* OpenAI-compatible API
* Good default for cost-sensitive production workloads on open-weight models

## SiliconFlow
SiliconFlow is a China-based inference provider with particularly strong, fast support for
Chinese-lab models (DeepSeek, Qwen, GLM, Kimi, etc.), often ahead of Western providers in adding
new releases.

* Frequently among the first providers to serve newly released Chinese models
* Aggressive pricing, often cited among the cheapest options for open-weight models
* Strong fit specifically when working with the Chinese model family covered in [Models](../models/README.md)
* Less established outside the Chinese-model ecosystem compared to Together/Fireworks/DeepInfra
