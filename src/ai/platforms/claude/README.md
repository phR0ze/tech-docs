# Claude <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Anthropic is an AI company founded in 2021 by former OpenAI researchers, including Dario Amodei and
Daniela Amodei. Claude is Anthropic’s family of large language models (LLMs). Claude powers
Anthropic's AI line of tools e.g:
* Claude chatbot
* Claude Code - coding assistant
* Claude Developer Platform - APIs

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
  - [Pricing](#pricing)
  - [Products](#products)
  - [Anthropic Academy](#anthropic-academy)
  - [Usage tips](#usage-tips)
  - [Console](#console)
  - [Claude Code](#claude-code)
- [Getting started](#getting-started)
  - [Install with NixOS](#install-with-nixos)
  - [Install with curl](#install-with-curl)
  - [First run configuration](#first-run-configuration)
- [Configuration](#configuration)
  - [Directory Structure](#directory-structure)
  - [.claude](#claude)
  - [CLAUDE.md](#claudemd)
  - [Status bar](#status-bar)
- [Sub-Agents](#sub-agents)
  - [Key traits](#key-traits)
  - [Rust dev](#rust-dev)
- [Commands and Skills](#commands-and-skills)
  - [init](#init)
- [Permissions](#permissions)

## Overview
You can setup a free account or use a paid plan. Enterprise plans will typically allow you to connect
via the familiar `Continue with Google` for access via OIDC.

**References**
* [Anthropic News](https://www.anthropic.com/news)

### Pricing
I do appreciate Anthropic's stance on keeping [Claude ad-free](https://www.anthropic.com/news/claude-is-a-space-to-think).
They have a really good write up stating:
```
We want our users to trust Claude to help them keep thinking—about their work, their challenges, and
their ideas. Our experience of using the internet has made it easy to assume that advertising on the
products we use is inevitable. But open a notebook, pick up a well-crafted tool, or stand in front of
a clean chalkboard, and there are no ads in sight. We think Claude should work the same way.
```

**Claude is free to try** but ***requires an account with a phone#***.
* Web, Android, IOS and desktop tools
* Generate code and visualize data
* Write, edit and create content
* Analyze text and images
* Search the web
* Create files and execute code
* Use desktop extensions
* Connect to Slack and Google workspace services
* Integrate with MCP tools
* Extended thinking for complex work
* Limits on the:
  * Number of messages you can send
  * Length of messages
  * Tool usage
  * Model choices
  * Artifact creation and usage

**Pro is $17/month or $200/year** adds on:
* More usage
* Includes Claude Code and Claude Cowork 
* Access to unlimited projects to organize chats and docs
* Access to research
* Memory across conversations
* Ability to use more claude models
* Claude in Excel

### Products
Anthropic has a number of AI powered tools including:
* Claude chatbot
* Claude Code - coding assistant
* Claude Developer Platform - APIs
* Claude for Chrome which can navigate, click and fill out forms in chrome. It can be leveratged from
  Claude Code, Cowork and Claude Desktop for end-to-end workflows.

Each of these tools have a different LLMs available to choose from:
* Opus
* Sonnet
* Haiku

### Anthropic Academy
Anthropic offers a number of courses through their Academy on how to use their technology.

**References**
* [Anthropic courses](https://anthropic.skilljar.com/)

### Usage Tips
How you prompt Claude will make a huge difference on how much usage you consume.

**References**
* [Claude usage limit practices](https://support.claude.com/en/articles/9797557-usage-limit-best-practices)

* Ask for markdown output to be put "in a code block" so you can copy out the raw characters and not
  the rendered version of markdown
* Refer back to previous information instead of repeating it
  * Use phrases like "As mentioned earlier" to build on earlier parts of the conversation and save
    token usage
* Batch similar requests in one message
  * If you have multiple relates tasks or questions, group them in a single message
    * e.g. instead of sending one math problem after the next, send them all
* Use project knowledge bases effectively
  * When you upload documents to a projecdt they are cached for future use
  * Only the new/uncached portions count against your limits in the future
  * Use projects for anything you'll reference multiple times
* Monitor your consumption
  * Pro, Max, Team or Enterprise plans allow you to view usage in `Settings >Usage`
* Coding tasks
  * Provide complete context about your coding environment in your initial message
* Writing assistance
  * Outline requirements, target audience, and key point comprehensively
  * Send entire text for editing in one message rather then breaking them up
* Research and anlaysis
  * Clearly define your research question and focus on areas initially
  * Provide all relevant data in a single, well-structured message

### Console
Claude Console is Anthropic's web portal for their AI tools. If you navigate out to the
[https://console.anthropic.com](https://console.anthropic.com) which will redirect to
[https://platform.claude.com/dashboard](https://platform.claude.com/dashboard) and click the `Log in`
button you'll then have the option to login with email and password or `Continue with Google` for an
OIDC flow. The Dashboard will give you options to:
* ***Create/generate prompts in the workbench***
* ***Create and manage Skills***
* ***Manage API keys***
* ***Usage analytics***
* ***Cost tracking***
* ***Account settings***

***Note: everyone has gravitated to the CLI though as the primary entry point***

**References**
* [Older walk through of Claude Console](https://www.youtube.com/watch?v=UfP5XWFVrww)

#### Workbench
The `Workbench` is a collection of options and tools for working with prompts. This is a richer
experience than the free ChatGPT or Claude's general chatbot interface. The Claude Workbench allows
for an ongoing interaction with Claude. You can name a prompt and do planning, thinking, research and
followup work all in that prompt that can then be referenced and reused later. Think of named prompts
as a project. You can also:
* Change the model being used
* Change the temperature of the AI
* You can set the max tokens to sample
* Work with different versions of your prompt

#### Generate prompt
***Generate prompt*** is an additional tool that can be used to enrich your original prompt with AI
suggestions to get the most out of your prompt from the AI. Its like the AI is guiding you on how to
ask your questions in the right way so that it can get you better answers.

Once your happy with your prompt you can click `Start Editing` to pull your prompt into the main
workbench space for interacting with prompts. This will give you a new prompt that you can name and
reuse as a project. You can then start filling in variables and other details.

### Claude Code
Claude Code can be installed and run directly in your terminal as a TUI application. It has a simple
command driven interface with help and prompts. The idea is that you'd run the `claude` code binary
from your project space, it would recognize that your running it in a programming project and you
could then interact with the AI agent as you would Copilot from the VSCode chat interface. The only
difference would be that there is no code editing window to show you changes.

I find the decoupling from a browser or the IDE to be freeing and exciting.

## Getting started
Claude Code can work in directly in your terminal to edit files, run commands and manage your entire
project from the command line.

### Install with NixOS
The upstream Claude Code installation for NixOS is slightly older at `2.1.39` while the curl bashing
approach provides the latest `2.1.45`, but isn't compatible with NixOS. So I've 
[rolled my own](https://github.com/phR0ze/nixos-config/tree/main/options/development/claude-code) to keep
up with latest:

### Install with curl
Super easy install method that gives you the latest version, but not compatible with NixOS.

1. Install with:
   ```bash
   $ curl -fsSL https://claude.ai/install.sh | bash
   ```

2. Add claude to your path
   1. On MacOS this will install to `~/.local/bin/claude`
   2. Run: `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc`

### First run configuration in CLI
1. Run with:
   ```bash
   $ cd your-project
   $ claude
   ```

2. Choose you're color theme

3. Now you'll have to login via your browser
   1. Choose `Anthropic Cosole account - API usage billing` for Enterprise accounts
   2. You'll be prompted in a browser to `Authorize` Claude Code connecting to your Anthropic account
   3. Back in the shell it will say `Login successful. Press Enter to continue`
   4. Continue through the security warning
   5. Choose `Yes, use recommended settings` for terminal setup
   6. Choose `Yes, I trust this folder` for security guide
   7. You now have Claude Code running in your terminal

### First run configuration in VSCode
On NixOS you have to install the Claude Code plugin then add the following to your configuration so
that VSCode can find the correct binary.

```json
{
  "claudeCode.claudeProcessWrapper": "/run/current-system/sw/bin/claude"
}
```

## Configuration
Claude Code doesn't follow the XDG Base Directory Support despite multiple filed issues out on
github. In the meantime configuration lives in:
* `~/.claude/` - user settings, scripts, memory
* `~/.claude.json` - global state (auth, MCP servers, theme)
* `.claude` - project level settings per repo

1. Run `/config`

### Directory structure
```
.claude/
├── CLAUDE.md                 # Main project context (required)
├── skills/
│   ├── skill-name/
│   │   ├── SKILL.md          # Skill definition and instructions
│   │   ├── scripts/          # Supporting scripts
│   │   └── references/       # Example implementations
├── agents/
│   └── security-reviewer.md  # Subagent definitions
├── hooks/
│   ├── pre-commit.sh         # Auto-execution scripts
│   └── post-edit.sh
├── settings.json             # Configuration
└── rules/                    # Domain-specific rules (optional)
   ├── api-conventions.md
   └── database-patterns.md
```

### .claude
Claude's configuration files either at `~/.claude` or in your repo at `.claude` are lazy loaded when
the directory is loaded starting with `CLAUDE.md` and only loading others as needed.

### CLAUDE\.md
The `CLAUDE.md` file is your 

### Status bar
The status bar is not enabled by default. To enable it run `/statusline` and let Claude go to town or
alternately if you don't trust Claude like me do:

1. Create `~/.claude/settings.json` to enable the statusLine
   ```json
   {
     "statusLine": {
       "type": "command",
       "command": "~/.claude/statusline.sh",
       "padding": 0
     }
   }
   ```

2. Create the file `~/.claude/statusline.sh`
   ```bash
   #!/usr/bin/env bash
   input=$(cat)
   
   MODEL=$(echo "$input" | jq -r '.model.display_name')
   COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
   PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
   
   COST_FMT=$(printf '$%.2f' "$COST")
   
   BAR_WIDTH=20
   FILLED=$((PCT * BAR_WIDTH / 100))
   EMPTY=$((BAR_WIDTH - FILLED))
   
   BAR=""
   [ "$FILLED" -gt 0 ] && BAR=$(printf "%${FILLED}s" | tr ' ' '█')
   [ "$EMPTY" -gt 0 ] && BAR="${BAR}$(printf "%${EMPTY}s" | tr ' ' '░')"
   
   echo "[$MODEL] $BAR $PCT% | $COST_FMT"
   ```

3. Make `~/.claude/statusline.sh` executable
   ```bash
   $ chmod +x ~/.claude/statusline.sh
   ```


## Sub-Agents
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

### Front
**Frontmatter** at the top of the agent file describes its purpose and provides basic identity
information.

| Field           | Purpose                                   |
|-----------------|-------------------------------------------|
| name            | Unique identifier                         |
| description     | When Claude should delegate to this agent |
| tools           | Allowed tools (e.g. Read, Bash, Grep)     |
| model           | sonnet, opus, haiku                       |
| disallowedTools | Tools explicitly denied                   |
| maxTurns        | Max agentic turns                         |

### Create Claude Code settings agent
Likely this agent persona will only be useful for a short time until I get my Claude Code environment
configured to my liking but so far I've spent a fair amount of time working with the
`~/.claude/settings.json` file and asking for assistance making tweaks. Wouldn't it be nice to have a
separate context thread and agent specializing in these changes.

* Code Reviewer
* Code Simplifier
* Security Reviewer
* Tech Lead
* UX Reviewer

### Rust dev

## Commands and Skills
Commands are an older format under `.claude/commands/` where each file is a simple markdown file with
no configuration options. `.claude/skills/` is a newer format that is directory based with a
`SKILL.md` file that supports YAML frontmatter for configuration options with more flexibility.
Regardless of capability through the function in a similar way surfaced as slash commands in the CLI.

Press `?` to get help printed out.

Slash commands provide shortcuts into pre-baked functionality in Claude.

### custom
You can create your own slash commands by simply creating markdown files in `~/.claude/commadns/`
e.g. `~/.claude/commands/markddown.md` with the following contents for its prompt will create a new
command `/markdown` that will convert the pasted code into a raw markdown table.

```
Convert this table to a raw markdown table inside a code block.
Align all pipe characters with padding, include a dashed header
separator row, and do not add trailing whitespace on any line.
```

**Usage**:
* Type out the `/markdown` command then paste in your table to convert
* Example: `/markdown [Pastedm text#3 +11 lines]`

### init
The `/init` command can be used to analyze your code base and create a new `CLAUDE.md` file. The
first thing Claude will do is recurse through your current directory to determine what kind of
project your working in. 

### clear
Claude Code has a context window, which is a limit on how much text it can hold in its memory. If you
consume that context window with random questions that don't pertain to the current work then you've
wasted some of Claude's available mental capacity for the task your trying to focus on. Additionally
random left overs from prior conversation will pollute the context and cause Claude to become
potentially confused or simply not have the mental capacity to answer your question as clearly as it
could have otherwise. 

Best practice
* `/clear` after every change in context

### compact

### rename

### resume
?

### plan
The `/plan` command is actually just automation the Claude provides to take your request and turn it
into a richer prompt that will generate actual research rather than just spitting out a precanned
response. It then takes this enriched prompt and passes it to a subagent of sorts. You can press
`Ctrl+o` to see it's reasoning.

**My Request**  
I like the notion of separating out my documentation into two categories: "docs/" for humans and
".specs" for AI. what do you think of this plan? provide constructive feedback on this approach and
reasons why it might be a good ide or not.

**Generated prompt**  
Research the following: I'm evaluating a documentation separation strategy where a developer uses
docs/ for human-readable documentation and .specs (dot-prefixed hidden directory) for AI-facing
specifications in software projects.

Please search the web and any available resources to find:
1. How Agent OS (buildermethods.com/agent-os) structures its spec files
2. Common patterns in spec-driven AI development (cursor rules, .cursorrules, CLAUDE.md conventions, windsurf rules, etc.)
3. Any existing conventions for separating human vs AI documentation in codebases
4. How Claude Code reads CLAUDE.md files and what kinds of content work best in them

Focus on practical patterns that have emerged in the community for structuring AI context files.

This triggered the AI to do a ton of web searching and legitimately good research.

### model

### context

### cost

### doctor

## Shortcuts

### Esc Esc
`Esc` `Esc` will undo the last action. This allows you to safely let Claude try something then roll
back before the change occured.

## Permissions
If at any point Claude desires to use terminal commands it will prompt you for permission.
You can respond with permission for this time only or permission for future similar commands in the
same directory.

### Persisting permissions
You can authorize Claude in a persistent way so that you don't always have to provide permission.
This can be done through the `~/.claude/settings.json` file for user wide permissions or
`.claude/settings.json` for project specific settings.

Its easier though to just use the `/permissions` command and have Claude add them e.g.
by simply specifying `WebFetch` we can allow any read only operations by default.

```json
{
  "permissions": {
    "allow": [
      "WebFetch",
      "WebSearch",
      "Bash(git *)"
    ]
  }
}
```


