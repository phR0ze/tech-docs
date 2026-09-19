# LiteLLM <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

LiteLLM is an open-source (`MIT`-licensed) [LLM router](../README.md) — a Python SDK and proxy
server, maintained by `BerriAI`, that translates calls to 100+ providers into a single
`OpenAI`-compatible format. Unlike a hosted aggregator like [OpenRouter](../openrouter/README.md),
LiteLLM is meant to be self-hosted: the proxy runs inside your own infrastructure and never sits in
the request path as a third party.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
- [Usage](#usage)
  - [SDK](#sdk)
  - [Proxy server](#proxy-server)
- [Self-hosting](#self-hosting)
  - [Docker Compose](#docker-compose)
  - [config.yaml](#configyaml)
- [Gotchas](#gotchas)

## Overview
LiteLLM supports 140+ providers and over 1,800 models. Code is written once against the `OpenAI`
SDK format; a config file, not application code, decides which provider a given model name
actually resolves to (`OpenAI`, `Anthropic`, `Vertex AI`, `Bedrock`, `Ollama`, etc.).

* Fully open source (`MIT`), self-hosted, and modifiable — request logs, provider API keys, and
  spend data stay in infrastructure you control
* Available as a lightweight Python SDK for direct in-process use, or as a standalone proxy server
  (the ***LLM Gateway***) for centralizing routing across a team
* Proxy adds virtual keys, budgets, rate limits, load balancing, automatic fallbacks, and an admin
  UI on top of routing itself
* Best fit when self-hosting is a requirement (compliance, data residency, no third party in the
  request path) rather than a preference

**References**
* [LiteLLM](https://www.litellm.ai)
* [LiteLLM Docs](https://docs.litellm.ai/docs/)
* [BerriAI/litellm on GitHub](https://github.com/BerriAI/litellm)

## Usage

### SDK
The Python SDK calls providers directly in-process, using the `OpenAI` `messages` format regardless
of the underlying provider:
```python
from litellm import completion

response = completion(
    model="deepseek/deepseek-r1",
    messages=[{"role": "user", "content": "hello"}],
)
```

### Proxy server
The proxy exposes the same `OpenAI`-compatible surface over HTTP, so any existing `OpenAI`-client
tooling can point its `baseURL` at it instead of embedding the SDK:
```bash
$ curl http://localhost:4000/chat/completions \
  -H "Authorization: Bearer $LITELLM_VIRTUAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek/deepseek-r1",
    "messages": [{ "role": "user", "content": "hello" }]
  }'
```

## Self-hosting

### Docker Compose
The published `docker-compose.yml` brings up the gateway on port `4000` plus a `Postgres` database
that stores models, virtual keys, and spend logs (`Redis` is added separately to coordinate rate
limits/budgets across multiple replicas):
```bash
$ curl -sSLO https://docs.litellm.ai/docker-compose.yml
$ docker compose up -d
```

`LITELLM_SALT_KEY` encrypts the provider API keys added through the admin UI — set it to a long
random value before adding any model you intend to keep, and never change it afterward (existing
keys become unreadable if it changes).

### config.yaml
Models, budgets, and routing behavior are defined declaratively and version-controlled alongside
the rest of the deployment:
```yaml
model_list:
  - model_name: deepseek-r1
    litellm_params:
      model: deepseek/deepseek-r1
      api_key: os.environ/DEEPSEEK_API_KEY

router_settings:
  routing_strategy: least-busy
  fallbacks: [{ "deepseek-r1": ["openai/gpt-4o"] }]
```
Mount it into the compose service and point the proxy at it with `--config=/app/config.yaml`.

## Gotchas
* Self-hosting means you own the operational burden (uptime, scaling, Postgres/Redis) that a
  managed aggregator like [OpenRouter](../openrouter/README.md) absorbs for you
* `LITELLM_SALT_KEY` rotation is a one-way trip — losing or changing it after keys are stored makes
  those provider keys unrecoverable
* Provider/model coverage is broad but self-managed — new upstream releases land whenever the
  config is updated, not automatically like a hosted aggregator's catalog
