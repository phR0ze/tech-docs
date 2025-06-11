# AI <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

Documenting various AI technologies

### Quick links
- [.. up dir](..)
* [Overview](#overview)
  * [Protocols](#protocols)
  * [Benefits](#benefits)
  * [Platform](#platform)
  * [Halucinations](#halucinations)
* [Popular AI Tools](#popular-ai-tools)
* [Prompt Engineering](#prompt-engineering)

### Linked pages
- [Copilot](copilot/README.md)
- [Duet](duet/README.md)
 
## Overview
AI is predicted to be a golden age of innovation rivaling Cloud, Mobile and possibly Internet:
* AI will touch every business in every sector
* AI is being democritized, meaning that it is available beyond just specialized situations and trained personel
* Google is investing BIG across everything with its new [Vertex AI platform](https://cloud.google.com/vertex-ai)

                               .----- AI Ecosystem ----.
                          ..../                         \....
                      Vertex AI                           Duet AI

### Protocols
* ***A2A (Agent2Agent)***
  * Google introduced A2A
  * Built to provide agents the ability to discover each other and negotiate tasks, exchange message 
    and collaborate across systems.
  * A2A allows agents to be part of a networked agentic team

* ***MCP (Model Context Protocol)***
  * Anthropic introduced MCP
  * Equips agents to be able to interact with external tools

* ***ADK (Agent Development Kit)***
  * Google introduced ADK
  * Purpose is to help developers build agents that can participate in A2A flows
  * ADK provides prebuilt components to help devs build agents faster

### Benefits
AI can reduce toil and improve efficiency
* Development
  * Enhance unit test coverage
  * Enhance documentation, summarize documents
  * Create content using GenAI (Email)
  * Automating business processes
  * Referenceability is key to building confidence with GenAI

### Platform
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

## Prompt Engineering
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

## AI Search
Allows for followup questions about the same material. You have a context to work from.

## Google Cloud Next 2023

### Vertex AI
Vertex AI is an enterprise ready generative AI. It allows you to take a snapshot of the Google LLMs 
and train and work with it in your own secure private space without leaking your information back to 
the whole.

* Vertex platform is where all your AI needs run
* You leverage Vertex AI for your own AI tools
* GKE Enterprise "new offering" for AI training performance
  * Mix of GPUs (A3 nvidia) and TPUs
* Vertex Model Garden
  * Ability to add third party models for use as well
  * Llama 2 and Claude 2 (third party) along side PaLM API for text (first party)
  * Tune and train on third party models while retaining control of your data
  * Grounding - tune AI responses for your enterprise data
    * e.g. question's about 401k once grounded will be specific to your company
  * Extensions - allows devs to integrate real-time data nad real-world actions
    * e.g. google built and partner built extensions for specific technologies
  * Tuning - different levels
    * Prompt design > Adapter tuning > Reinforcement > Full fine tuning 
* New AI tools based on Vertex AI for generating applications for your use cases
  * Simply describe the requirements and AI builds it for you

### Chirp
Speech to text model

### Codey
Text to code generation on Vertex AI

  * Gitlabs is using this

### Imagen
Text to image generation on Vertex AI

  * Provides Digital Watermarking for AI generated images in a way that is invisiable to the human eye
  * Verification tools can be used to search for water marks to see if was generated by AI

### Notes
* Workload Optimized Infrstructure
  * 

* Duet AI
  * AppSheet for Google Chat
    * "I need to capture incident reports. Each incident has a type, a status, a description, and a 
    date. Each incident should be assigned to a team member"
    * "Type should be major or minor"
    * Create App
  * Unstructured audio capture with Duet AI
  * Generate code
    * Describe code to be generated
    * Select generated code and say "write unit tests"

* Cloud Run Jobs
  * Run to completion capability for background work
  * It hides all the infra complexity (serverless)
  * When not running, costs nothing (serverless)

* Complexity for shipping code
  * How many engineers are required for a simple boolean change?
  * How long does it take to build and deploy?
  * Solve by empowering vertical and horizontal ownership
  * Building extensible but shared infrastructure
    * One over everything: one workload identity, one network, one ...
  * Use open source tools and APIs where possible
  * Platform and product architecture enables development and release velocity whether it's through 
  pluggability or portability or use of shared services our goal is to deliver the best functionality 

* `Cloud Build` and `Cloud Deploy` for CI and CD
  * Just added console UI support showing pipelines
  * Promoting images in UI, canary deployments etc...
  * How to application respond to scaling events
  * Infra is easy to maintain to spend more time adding value than keeping the lights on
  * Cloud Run performance 1 always running instance, CPU boost
  * Assured OSS checks

* Secure Software Supply Chain (S3C)
  * How are we securing out developer env
  * How are we ensuring the integrity of the build process

* Cloud Workstations
  * vscode in the browser
  
* Jump Start Solutions
  * Reduce toil by shifting it down into the platform
  * Jump start provides a one click to deploy the resources needed automatically
  * Shows products used, architecture, guided step by step guide and reference docs
  * Jump start solutions are open source and available on github

<!-- 
vim: ts=2:sw=2:sts=2
-->

