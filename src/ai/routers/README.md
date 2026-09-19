# Routers <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

An overview of LLM routers — also called model aggregators or LLM/AI gateways — with a comparison
of the top competitors in the space.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
- [OpenRouter](#openrouter)
- [LiteLLM](#litellm)
- [Portkey](#portkey)
- [Helicone](#helicone)
- [Cloudflare AI Gateway](#cloudflare-ai-gateway)

### Linked pages
- [OpenRouter](openrouter/README.md)
- [LiteLLM](litellm/README.md)

## Overview
An ***LLM router*** sits between an application and one or more LLM providers, exposing a single,
usually `OpenAI`-compatible API in front of them. Instead of managing separate SDKs, API keys, and
billing accounts per provider (`OpenAI`, `Anthropic`, `Google`, self-hosted open-weight models,
etc.), a caller sends every request to the router and it handles model selection, routing, and
fallback behind the scenes.

The category goes by a few interchangeable names depending on which angle a vendor is emphasizing:
* ***LLM router*** — emphasizes the routing/model-selection function
* ***Model aggregator*** — emphasizes unifying many providers behind one catalog
* ***LLM gateway*** / ***AI gateway*** — emphasizes the infrastructure/control-plane role (auth,
  rate limits, observability, governance) alongside routing

None of these are model providers themselves — they don't train or host models — they're
infrastructure that sits in the request path between a caller and the labs in
[Providers](../providers/README.md) and [Models](../models/README.md).

## OpenRouter
A hosted, managed aggregator with the broadest model catalog of any router — one API key against
400+ models from 60+ providers, an `OpenAI`-compatible API, and automatic fallback between upstream
providers. See [OpenRouter](openrouter/README.md) for full details.

## LiteLLM
An open-source (`MIT`) Python SDK and proxy server that translates calls to 100+ providers into a
single `OpenAI`-compatible format. Unlike OpenRouter it's meant to be self-hosted — you run the
proxy in your own infrastructure and it never sits in the request path as a third party. See
[LiteLLM](litellm/README.md) for full details.

## Portkey
Portkey is a full-stack LLMOps platform that bundles an AI gateway with observability, guardrails,
governance, and prompt management on top of it.

* Open-source gateway core routing across 1,600+ models, with a paid platform layered on top
* Built-in guardrails, budget/access governance, and prompt management aimed at regulated,
  enterprise environments
* Best fit when governance and compliance tooling matter as much as raw routing
* Heavier to adopt than a router-only tool if all you need is model access

## Helicone
Helicone combines an open-source LLM gateway with request logging, cost tracking, and monitoring
built in from the start, rather than as an add-on.

* Open-source, with both a hosted and self-hosted option
* Routing is paired tightly with observability — logging, cost, and latency tracking are first-class
* Best fit for low-latency open-source routing where built-in observability matters more than
  catalog breadth
* Narrower provider/model catalog than aggregators like OpenRouter or Requesty

## Cloudflare AI Gateway
Cloudflare's AI Gateway routes AI traffic through Cloudflare's existing edge network, layering
caching, rate limiting, and analytics on top of requests to providers like `OpenAI`, `Anthropic`,
and `Hugging Face`.

* Runs on Cloudflare's global edge, with a free tier and usage-based pricing beyond it
* Edge caching, rate limiting, request logging, and analytics out of the box
* Integrates with Cloudflare Workers for custom pre/post-processing logic on requests
* Best fit for teams already built on Cloudflare's stack; less compelling as a standalone choice
  otherwise

**References**
* [Best LLM Gateways for Developers](https://www.braintrust.dev/articles/best-llm-gateways-2026)
* [Best LLM Routing Platforms Compared](https://www.requesty.ai/blog/best-llm-routing-platforms-compared-2026-requesty-portkey-litellm-openrouter)
