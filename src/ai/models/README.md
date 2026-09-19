# Models <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

A comparison of notable LLMs, grouped by origin, with a short background on each model and its
standout capabilities.

### Quick Links
- [.. up dir](..)
- [Chinese LLMs](#chinese-llms)
  - [DeepSeek](#deepseek)
  - [Qwen](#qwen)
  - [GLM](#glm)
  - [Kimi](#kimi)
  - [MiniMax](#minimax)
- [Model Comparisons](#model-comparisons)
  - [Coding agent harness](#coding-agent-harness)

## Chinese LLMs
Chinese AI labs have converged on open-weight releases as a competitive strategy, trading direct
API revenue for adoption and mindshare. Most of the models below ship under permissive licenses
(MIT or Apache 2.0) and are available self-hosted, via the vendor's own API, or through aggregators
like OpenRouter.

**Open-weight vs. open-source** — "open-weight" means the trained parameters are published for
anyone to download, self-host, fine-tune, or build on top of. It does not mean "open-source": the
training data, training code, and full pipeline behind those weights remain proprietary, so the
model can't be reproduced or audited at the source. None of the models below meet the stricter
open-source bar, even under a permissive weight license (`MIT`, `Apache 2.0`) with no usage
restrictions.

### DeepSeek
Spun out of the quant trading fund High-Flyer, DeepSeek made its name by training frontier-grade
models at a fraction of the usual compute cost. Its `V3.x`/`V4` line targets general-purpose chat
and coding, while the `R1` line is a dedicated reasoning model trained with reinforcement learning
to "think" step by step before answering.

* Mixture-of-experts architecture (e.g. 671B total / 37B active params) keeps inference cheap
  relative to model size
* `R1` line is purpose-built for reasoning and competes with closed reasoning models like OpenAI's `o1`
* MIT-licensed open weights, widely supported by self-hosting tools (Ollama, vLLM)
* Best price-to-performance ratio of the major labs; frequently the cheapest frontier-tier tokens
  on OpenRouter
* Free `:free` variants have historically been available on OpenRouter, though availability rotates

### Qwen
Alibaba's Qwen family is the most widely adopted open base model globally, spanning an unusually
broad range of sizes (from ~0.5B up to 400B+ parameters) and modalities (text, vision, audio, code).
Its scale of adoption makes it a default choice for local/self-hosted deployment.

* Apache 2.0 license with the broadest size lineup of any lab, from edge-sized to frontier-scale
* `Qwen3` introduced a hybrid "thinking" vs "non-thinking" mode, letting one model trade off
  latency against reasoning depth
* `Qwen3-Coder` is a coding-specialized variant with strong agentic tool-calling and top SWE-bench
  scores; became the most-downloaded open coding model by January 2026
* Strong multilingual support, reflecting Alibaba's global commercial footprint
* Widest ecosystem/tooling support of any Chinese lab (quantizations, fine-tunes, inference engines)

### GLM
Developed by Zhipu AI (a Tsinghua University-affiliated lab that spun off its international arm as
Z.ai), the GLM/ChatGLM lineage has pivoted toward large-context, agentic coding use cases with its
`GLM-5.x` generation.

* `GLM-5.2` is a 753B-parameter MoE model (~40B active) with a 1M-token context window and up to
  131K tokens of output
* Leads independent coding/agent leaderboards, with a verified Code Arena Elo around 1,530
  reflecting real developer head-to-head preference
* `GLM-5.1` ships under an MIT license
* Popular as a backend model for agentic coding harnesses (OpenCode, Claude Code-style tools)
* `z-ai/glm-5.2:free` has been available as a free tier on OpenRouter

### Kimi
Moonshot AI built its early reputation on ultra-long-context chat, then pivoted hard into agentic
coding models with the `K2`/`K3` line. Its distinguishing trait is stability over very long,
multi-step agent sessions rather than single-turn benchmark scores alone.

* `K3` is a ~2.8-trillion-parameter MoE model, one of the largest open-weight releases to date
* `K2.6` sustained 4,000+ tool calls over a 13-hour uninterrupted agent session in published
  benchmarks — a stability ceiling other open models haven't matched
* Leads SWE-Bench Pro among open models (~58.6%) and scores strongly on SWE-Bench Verified (~80.2%)
* Earlier `k1.5` line was notable for multimodal tasks with sub-500ms response times
* No free API tier — pay-per-token only (~$0.95/M input, ~$4.00/M output), though the weights
  themselves are open for self-hosting

### MiniMax
A Shanghai-based startup backed by Alibaba and Tencent, MiniMax has increasingly shown up alongside
Kimi/GLM/Qwen in coding and agentic benchmark comparisons with its `M2.x` line, positioning itself
as a lower-cost alternative rather than an outright frontier leader.

* MoE architecture tuned for cost-efficient agentic workflows rather than peak benchmark scores
* Competitive but generally a step behind Kimi/GLM/Qwen in head-to-head coding comparisons
* Part of the broader trend of Chinese labs pushing aggressive free/near-free pricing on
  OpenRouter to capture developer mindshare
* Multimodal roadmap extending beyond text, including earlier speech/voice models

## Model Comparisons

### Coding agent harness
Comparing `Kimi`, `Qwen`, and `GLM` specifically for driving a coding agent harness through a long,
multi-step, tool-heavy workflow (e.g. an APK RE/recompile loop: `apktool`/`jadx` → edit →
`gradle`/`apktool` build → `zipalign` → sign → retest, repeating on failures):

1. **Kimi K2.6** — the standout pick. Its distinguishing benchmark is agentic stability: 4,000+ tool
   calls sustained over a 13-hour uninterrupted session, plus 80.2% SWE-Bench Verified and 58.6%
   SWE-Bench Pro (best of the group). Long, multi-step, tool-heavy workflows are exactly the case
   where a model that degrades over long sessions will drift or forget earlier constraints partway
   through — K2.6 doesn't.
2. **Qwen3-Coder(-Next)** — close second. Efficient MoE (only ~3B active params), 256K native
   context, Apache 2.0, and the most battle-tested in the wild (most-downloaded coding model as of
   January 2026) — strong for the raw code-editing/smali-diffing part, slightly less proven on very
   long agentic sessions than Kimi K2.6.
3. **GLM-5.1/4.6** — good agentic front-end/dev preference (Code Arena Elo 1,530) but less
   specifically benchmarked for sustained tool-call marathons.

If you're building an actual pipeline (a harness driving `apktool`/`jadx`/`gradle`), Kimi K2.6 is
the better bet specifically because of tool-call endurance; if you want something leaner to
self-host, Qwen3-Coder-Next.

**References**
* [Kimi K2.6 vs GLM 5.1 vs Qwen 3.6 Plus vs MiniMax M2.7: Which Open Source Model Wins for Coding in 2026 - Atlas Cloud Blog](https://www.atlascloud.ai/blog/guides/kimi-k2-6-vs-glm-5-1-vs-qwen-3-6-plus-vs-minimax-m2-7-coding-2026)
* [Best Open-Source Coding Model 2026: Kimi K3 vs GLM-5.2 vs DeepSeek V4 vs Qwen3 | Morph](https://www.morphllm.com/best-open-source-coding-model-2026)
* [Qwen3-Coder-Next Technical Report](https://arxiv.org/html/2603.00729v1)
* [Kimmy K2.6 and Qwen 3.6: The Open-Source Models Closing the Frontier Gap | MindStudio](https://www.mindstudio.ai/blog/kimmy-k2-6-qwen-3-6-open-source-frontier-models)
