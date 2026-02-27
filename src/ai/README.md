# AI <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

AI technology is moving fast.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
  - [Primitives](#primitives)
  - [Protocols](#protocols)
  - [Halucinations](#halucinations)
  - [Progressive disclosure](#progressive-disclosure)
- [Popular AI Tools](#popular-ai-tools)
- [Prompt Engineering](#prompt-engineering)

### Linked pages
- [Agents](agents/README.md)
- [Context Engineering](context_engineering/README.md)
- [Platforms](platforms/README.md)
- [Skills](skills/README.md)
 
## Overview
AI is predicted to be a golden age of innovation rivaling Cloud, Mobile possibly Internet:
* AI will touch every business in every sector
* AI is a composition of LLMs (i.e. the model) and agents (i.e context, pompt and tools)
* Engineers, with their rigorous conditioning in critical thinking, are potently positioned to orchestrate AI
* AI is being democritized, meaning that it is available beyond just specialized situations and trained personel

Foundationally AI is built on top of ***Large language models*** which are:
  * ML algorithms that can recognize, predict, and generate human languages
  * Pre-trained on petabyte scale datasets of text, images, video, chat, audio etc...
  * Domain specific models
    * Sec-PaLM - cybersecurity
    * Med-PaLM - life science and healthcare

AI work loads are pushing hyperscalers to develop specialized hardware and chips to handle the 
demanding AI workloads. `TPUs` are specialized processors for machine learning calculations and 
require more traffice to learn at first but later are more impactful with less power consumption.

TPUs are 15-30 times faster than current GPUs for AI workloads

### Primitives
There are 4 core primitives in AI engineering: `Context`, `Model`, `Prompt`, `Tools`. An AI agent
provides the context, prompt and tools to leverage the model.

### Protocols
There are a few protocols out there for interacting with LLMs, but Anthropic's MCP won the battle for
dominance.

* ***MCP (Model Context Protocol)***
  * Anthropic introduced MCP
  * Equips agents to be able to interact with external tools

### Halucinations
AI has been plagued with what they call AI halucinations, in that you haven't been able to trust the 
answers you get back from AI from an accuracy or safety perspective. Often generative AI will just 
make up answers. So everything is disclaimed with caveats to check AIs work.

This will get better over time but will likely always be a thing.

## Popular AI Tools

| AI Tool         | Category
| --------------- | ---------------
| Github Copilot  | Coding
| Canva           | Image
| ChatGPT         | Conversational
| Fathom          | Meeting
| Gemini          | Multimodal
| Grammerly       | Writing
| Murf.ai         | Voice
| Notion          | Productivity
| Synthesia       | Video
| Zapier          | Automation

### Progressive disclosure
To protect context skills are loaded in a progressive way.
1. Metadata
   * Name: 64 chars
   * Description: 1024 chars
2. Full instructions
   * should be < 5k tokens
3. Linked files
   * additional resources loaded only if needed

## Prompt Engineering
The primitives in te LLMs understand natural language well enough
In order to get the most out of AI you need to be good and coaxing AI to respond the way you want it 
to. This is the art of prompt engineering.

* We are ***WE ARE***
* You are a ***CHARACTER*** called ***NAME***
* As a chat bot, your mission is ***MISSION***
* The customer profile is the following ***PROFILE***
* Use this profile information to answer the questions without sharing details
* The conversation you have already with the customer is the following ***HISTORY***
* Based only on the following information ***CONTEXT***
* Don't assume anything
* Answer the question below ***QUESTION***
* Answer what's important. Be consice ***PERSONALIZATION***
* If you are not sure then answer that you don't have enough information and hand over to human agent
