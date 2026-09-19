# OpenRouter <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

OpenRouter is a hosted, managed [LLM router](../README.md) — one API key gives access to models
from dozens of underlying providers through a single `OpenAI`-compatible endpoint, with automatic
routing and fallback if an upstream provider is degraded or rate-limited.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
- [Pricing](#pricing)
- [Usage](#usage)
  - [Model selection](#model-selection)
  - [Free models](#free-models)
- [BYOK](#byok)
- [Gotchas](#gotchas)

## Overview
Founded in early 2023 to address the fragmented LLM landscape, OpenRouter aggregates 400+ models
from 60+ providers behind one catalog. It is not a first-party inference host itself — for most
models it routes the request to one of several underlying providers (many of the same ones covered
in [Providers](../../providers/README.md)) and returns the response.

* Broadest model catalog of any router — one key, hundreds of models across every major lab
* `OpenAI`-compatible API, making it a drop-in `baseURL` swap for most existing tooling
* Automatic fallback/routing if a given upstream provider is degraded or rate-limited
* Best fit when maximum model choice and minimal setup matter more than raw latency or governance

**References**
* [OpenRouter](https://openrouter.ai)
* [OpenRouter Models](https://openrouter.ai/docs/guides/overview/models)

## Pricing
OpenRouter doesn't mark up the underlying model's list price — it makes money on credit purchases
and BYOK usage instead:
* ***5.5%*** fee (`$0.80` minimum) on card credit top-ups, ***5%*** on crypto top-ups
* A self-serve ***Business*** tier charges an ***8%*** platform fee on credit purchases in exchange
  for inference locked to providers inside the EU or US only
* [BYOK](#byok) usage is free up to a monthly list-price usage allowance, then billed a smaller fee

## Usage

### Model selection
Models are addressed as `<provider>/<model>` strings passed in the standard `OpenAI`-style request
body, e.g. `openai/gpt-4o` or `deepseek/deepseek-r1`. `openrouter/auto` is a meta-router that picks
a model automatically based on the prompt rather than requiring an explicit choice.

```bash
$ curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek/deepseek-r1",
    "messages": [{ "role": "user", "content": "hello" }]
  }'
```

### Free models
Many models expose a `:free` variant (e.g. `z-ai/glm-5.2:free`) that routes to a free-tier upstream
where available. Availability rotates and is not guaranteed — useful for prototyping without cost,
not for production traffic that needs to stay up.

## BYOK
***BYOK (Bring Your Own Key)*** lets a caller attach their own provider API keys so inference bills
the provider account directly, while OpenRouter still handles routing, logging, and fallback.

* Free up to `$25,000`/month of list-price inference (`$200,000`/month on Enterprise), then a `5%`
  fee on top of what the same model would normally cost
* Useful when a provider relationship (pricing tier, compliance terms, dedicated capacity) already
  exists and only the routing/fallback layer is wanted, not OpenRouter's own billing

## Gotchas
* Adds measurable routing overhead (commonly cited in the `40`-`55ms` range) versus calling a
  provider directly — a factor for latency-sensitive workloads
* Not a first-party inference host — no fine-tuning, custom deployments, or dedicated capacity;
  see [Together AI](../../providers/README.md#together-ai) or [Fireworks AI](../../providers/README.md#fireworks-ai)
  if that's needed instead
* Routing to a degraded upstream provider is still possible; check provider status if a specific
  model behaves inconsistently
