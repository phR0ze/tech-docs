# Spec Driven Dev <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Spec-Driven Development (SDD) with AI is a methodology with specifications being the primary artifact
of the engineer, not code. Before ever prompting for AI to generate code a developer first pursues
the creation with AI assistance of a specification that will act as a referenceable blueprint from
which you and/or AI can work as you jointly build out your project. This artifact becomes the north
star from which everything is gauged and provides a repeatable cohearant way to incrementally build
out a cohesive solution.

**Thus you:**
* Write specs first and reference them throughout workflow
* Maintain specs as living documentation keep fresh with the latest changes and learnings

This allows for questions such as "according to our spec what is next on deck to build out?".

### Quick Links
- [.. up dir](..)
* [Overview](#overview)
  * [Why it works well with AI](#why-it-works-well-with-ai)
  * [Major SDD Frameworks](#major-sdd-frameworks)
  * [Practical Contrast](#practical-contrast)
  * [Where It Shines Most](#where-it-shines-most)
  * [Specs location](#specs-location)
  * [Modified PRD](#modified-prd)
* [Standards](#standards)

## Overview
The core SDD pattern consists of:
* Human/AI writes spec > AI reads spec > AI generates code > Human/AI validates against spec > Iterate
* The underlying principle is that the human's highest-leverage contribution is clarity of intent. A
  spec externalizes that intent into a durable, reviewable artifact — which AI can then execute
  faithfully and repeatedly.

The spec typically includes:
- What the system should do (behavior, not implementation)
- Inputs and outputs with types, shapes, and constraints
- Edge cases and error conditions
- Acceptance criteria — how you'll know it's correct
- Architectural decisions already made by the human

**References**
* [Rasmus Wiiding's PRP](https://github.com/Wirasm/PRPs-agentic-eng)
* [Cole Medin's context engineering](https://github.com/coleam00/context-engineering-intro)
* [Brian Casel's Spec-Driven Development](https://buildermethods.com/agent-os)

### Why it works well with AI
Modern AI now has the ability to write their own specs well with `/plan` mode.

1. ***Better context, better output*** - AI models generate code that matches their input fidelity.
   Vague prompts produce vague code. A precise spec gives the model enough signal to produce precise
   implementations.

2. ***Reduced hallucination and drift*** - When the AI has a concrete spec to anchor to, it's less
   likely to invent requirements or make assumptions that diverge from your intent. It has something
   to be faithful to.

3. ***Separates thinking from generating*** - Writing the spec forces you to solve the design problem
   before the implementation problem. This is the same principle as rubber-duck debugging — the act
   of articulating the problem clearly often reveals the solution.

4. ***Built-in verification*** - The spec doubles as a test oracle. After generation, you (or the AI)
   can check: does this implementation satisfy each criterion in the spec? This makes review much
   more structured than asking "does this look right?"

5. ***Human retains decision authority*** - The AI handles mechanical translation from spec to code.
   Architectural choices, tradeoffs, and business logic remain in the spec — authored by you. This
   prevents the AI from making consequential decisions implicitly.

6. ***Documentation as a byproduct*** - The spec is already documentation. You don't have to write it
   separately after the fact.

7. ***Iteration is precise*** - When something is wrong, you update the spec and regenerate, rather
   than trying to describe a delta in natural language. "Change the spec at line 12 to require X" is
   more reliable than "actually, I also need it to handle X."

### Major SDD Frameworks
The AI development community is gradually unifiying behind SDD in some form or another. More and more
of this pattern is being incorporated directly into AI CLI tools e.g. Claude's `/plan` command which
facilitates spec generation.

Likely what we'll see is that the need for external scaffolding 

***GitHub Spec Kit — github/spec-kit***
- Workflow: Constitution → Specify → Plan → Tasks → Implement
- Uses a `Constitution.md` to establish non-negotiable architectural principles
- CLI-based scaffolding, integrates with Copilot, Claude Code, Cursor, Gemini CLI
- Best for: Greenfield projects, team environments

***Amazon Kiro — kiro.dev***
- Agentic IDE (not just a plugin): natural language → user stories → technical design → code
- Autonomous agents trigger on file save events (background docs, tests, optimization)
- Real impact: Drug discovery agent built in 3 weeks; life sciences teams going months → weeks
- Best for: Full-stack production systems

***Tessl — tessl.io ($125M funded)***
- Most ambitious: pursuing true spec-as-source (humans only edit specs, never code)
- Includes a Spec Registry with 10,000+ pre-built specs for common libraries
- Addresses AI hallucinations by anchoring agents to formal, versioned specs
- Best for: Teams ready for high-commitment spec-as-source workflow

***BMAD-METHOD (Breakthrough Agile AI-Driven Development)***
- Multi-agent: 19+ specialized AI personas (Analyst, PM, Dev, Security, etc.)
- Each agent has a defined role; documentation is the source of truth
- Enterprise case study: 55% reduction in Basel III compliance implementation
- Best for: Complex enterprise projects with large teams

***GSD (Get Shit Done) — glittercowboy/get-shit-done***
- Intentional reaction to BMAD/Spec Kit complexity: 3 phases, 2 prompts, 1 loop
- Used by engineers at Amazon, Google, Shopify, Webflow
- Solo dev benchmark: 100,000 lines of code in 2 weeks
- Best for: Individuals, small teams wanting SDD without heavy process

***OpenSpec — Fission-AI/OpenSpec***
- Lightweight, brownfield-first (designed for existing codebases, not just greenfield)
- YAML/JSON/Markdown, no API keys required, works with 20+ tools
- Best for: Teams wanting a low-friction entry point to SDD

### Practical Contrast

| Ad-hoc prompting                      | Spec-Driven                                                                   |
| ------------------------------------- | ----------------------------------------------------------------------------- |
| "Write a function that parses dates"  | Spec defines: input format, output format, locales, error behavior, examples  |
| AI guesses your intent                | AI has explicit criteria to meet                                              |
| Hard to know when you're done         | Spec acceptance criteria define "done"                                        |
| Review feels subjective               | Review is: does it match the spec?                                            |
| Requirements drift silently           | Spec changes are explicit and visible                                         |

### Where It Shines Most

- Complex features with many edge cases
- APIs and interfaces that other code depends on
- Security-sensitive code where correctness is critical
- Team environments where specs communicate intent to both humans and AI
- Long-running projects where you need to regenerate or modify code later

### Specs location
Including `specs/` inside `.claude` obfuscates them from human consumption as documentation
artifacts. Thus the proposal would be to use `docs/specs/` as the location for specification
documentation that would be shared by both human and AI. It would then be clearly visible and
intuitive to either human or AI and be a single location to update. This would mean that AI
directives stored in `.claude` would need to reference `docs/specs/` as needed.

```
project/
├── CLAUDE.md               # Entry point: imports + key conventions (target: < 200 lines)
├── docs/
│   ├── PRD.md              # Product Requirements Document
│   └── specs/              # Feature specs — AI-structured, human-readable
│       ├── _template.md    # Canonical spec template
│       ├── auth.md         # Example auth specification
│       └── payments.md     # Example payments specification
└── .claude/                # Claude Code project config
    ├── commands/           # Project-specific slash commands
    └── settings.json       # Project-specific settings
```

### Modified PRD
A document that describes what a product or feature should do, written before development begins. It
typically covers the following but we'll modify it to also include ***implementation/validation***
guiance.

- ***Purpose*** — the problem being solved and why
- ***User stories / use cases*** — who uses it and how
- ***Functional requirements*** — what the system must do
- ***Non-functional requirements*** — performance, security, scalability constraints
- ***Out of scope*** — explicitly what is not being built
- ***Success metrics*** — how you'll know it worked

## Standards
Anything that you'd need to re-teach Claude repeatedly belongs here:

* `~/.claude/CLAUDE.md` - user level AI guidance, YOUR standards, preferences, workflows
* `~/Projects/myapp/CLAUDE.md` - app level conventions
* `~/Projects/myapp/frontend/CLAUDE.md` - director level subsystem specific rules

### Content that works best
Research and community consensus show these sections are most effective:

Include (High-Leverage):
- Bash commands Claude can't guess (build, test, deploy commands with exact flags)
- Code style rules that differ from language defaults (ES modules vs. CommonJS, import patterns)
- Testing instructions and preferred test runners
- Repository etiquette (branch naming, PR conventions)
- Architectural decisions specific to your project
- Developer environment quirks (required env vars, database setup)
- Common gotchas or non-obvious behaviors
- Project-specific warnings

Exclude (Noise):
- Anything Claude can figure out by reading code
- Standard language conventions Claude already knows
- Detailed API documentation (link to docs instead)
- Information that changes frequently
- Long explanations or tutorials
- File-by-file codebase descriptions
- Self-evident practices ("write clean code")

Length & Performance
- Recommended: < 300 lines (shorter is better)
- Problem: If `CLAUDE.md` is too long, Claude ignores half because important rules get lost
- Rule of thumb: For each line, ask "Would removing this cause Claude to make mistakes?" If not, cut it

### References
AI context can be quickly consumed just by conventions, best practices and standards. To avoid this
you'll want to break it down to only include what is needed for the given situation. You can
accomplish through references in your top level doc.

### Docs

**References**
* [Spec-driven dev](https://github.com/gotalab/cc-sdd)

Separate out AI from human documentation:
* .ai - for AI documentation
* docs/ - for human documenation

### Benefits
* ***Spec driven*** - Approve requirements/design upfront, then AI implements exactly as specified
* ***Memory*** - AI remembers your architecture, patterns, and standards across sessions
* ***Agent teams*** - Same spec-driven process across Claude, Cursor, Gemini, Codex, Copilot, Qwen, OpenCode, Windsurf
* ***Parallel execution*** - Tasks decomposed for concurrent implementation with dependency tracking

### Industry
* Cursor `.cursorrule`
* Windsurf `.windsurf/rules`
* Kiro
* Codex
* GitHub Copilot
* Claude Code
* Gemini CLI

### Persistence
Keeping the AI context primed with the correct knowledge but as focused as possible is the key. To
accomplish this we need to perist context.

### Agent OS example
The work that Brian Casel did around some early pioneering of more prescriptive frameworks, i.e.
harnesses, from which an AI can gather context and perform more effectively. Agent OS creates specs
with folder names auto-generated as `YYYY-MM-DD-HHMM-{feature-slug}/`. This creates a persistent,
time-versioned record that survives beyond individual conversations.

```
./agent-os/
├── standards/                # Project-specific coding standards
├── specs/                    # Timestamped specification folders
│   ├── 2026-01-15-1430-user-comment-system/
│   │   ├── plan.md           # Full implementation plan
│   │   ├── shape.md          # Scope, decisions, context
│   │   ├── references.md     # Pointers to similar code
│   │   ├── standards.md      # Relevant standards content
│   │   └── visuals/          # Mockups, screenshots
│   └── 2026-01-10-0900-api-refactor/
└── product-docs/             # Product-level documentation
```


