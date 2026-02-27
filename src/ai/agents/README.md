# Sub-Agents <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

### Quick Links
- [.. up dir](..)
* [Overview](#overview)
  * [Key traits](#key-traits)
  * [Frontmatter](#frontmatter)
* [Extract text from image](#extract-text-from-image)

## Overview
Sub-agents located at `.claude/agents/` are best for delegating isolated tasks that would clutter
your current context window. They are lightweight, specialized workers spawned within a single
session. The user or Claude can delegate work to the sub-agent which runs in its own isolated context
and returns the summarized results back to the main instance.

### Key traits
* Each gets its own context window (isolated from the main conversation)
* Tool access can be restricted per agent (e.g. read-only reviewer)
* Results are summarized back to the main conversation
* Cannot nest (sub-agents can't spawn sub-agents)
* Lower token cost as it stays within a single session's budget
* You can define your own custom sub-agents in `.claude/agents`

### Frontmatter
**Frontmatter** is the term used to describe the header of an agent file which is its own segmented
document separated from the body document with well known fields. It describes the agent's purpose
and provides basic identity information, configuration and when to use it.

| Field           | Purpose                                   |
|-----------------|-------------------------------------------|
| name            | Unique identifier                         |
| description     | When Claude should delegate to this agent |
| tools           | Allowed tools (e.g. Read, Bash, Grep)     |
| model           | sonnet, opus, haiku                       |
| disallowedTools | Tools explicitly denied                   |
| maxTurns        | Max agentic turns                         |

## Settings agent
Extracting text from an image would be a fantastic tool to create as both a skill to be able to be
used in one offs or as a sub-agent to avoid poluting my context.


