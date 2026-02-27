# Stategies <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

AI guidance approaches are a dime a dozen at this point. One thing is certain though `context is
king`. LLMs are stateless and thus you need to carefully engineer your context input to produce
better output from the LLM. Additionally the context can't be overly poluted or LLM effectiveness
plummets leading to the `Dumb Zone` at context consumption of anything over 40% typically.

**Goals**
* Compression of intent
* Reliable execution
* Don't outsource the thinking

### Quick Links
- [.. up dir](..)
* [Overview](#overview)
* [On-demand context](#on-demand-context)

### Linked pages
- [Spec Driven Dev](spec_driven_dev/README.md)

## Overview
Focusing on context engineering we currently have two patterns emerging. `Spec-Driven Development` is
one gaining momentum but those on the frontier are already pulling away it with the notion that the
term is semantically diffusing to mean too many things to be useful anymore. The second pattern
emerging is more about tuning the first really in my opinion. I would call it the `on-demand context`
approach which acknowledges the necessity for careful context priming, but leans on on-demand methods
to gather than rather than persisting it as specification documents for later retrieval.

### Planning
As you planning goes up your reliability of execution goes up but your readability goes down.

## On-demand context
Dynamically on-demand start with building a plan about the codebase  `/research_codebase` "I'm
implementing a new feature related to SCM providers and how they integrate with Jira, Linear, etc -
can you tell me everything about how these systems work today?"

1. Research
2. Plan
3. Implement

**References**
* [Engineering the future of AI](https://www.youtube.com/watch?v=rmvDxxNubIg)

### Benefits of on-demand
* Actual truth based on the code itself not outdated documents
* 
