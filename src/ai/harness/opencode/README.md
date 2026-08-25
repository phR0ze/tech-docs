# OpenCode <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

OpenCode is an open source AI CLI harness similar in nature to Claude Code CLI or Gemini CLI. Given
that its open source it has some niceties that the others don't and is missing some features that the
others have.

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
- [Tips](#tips)
  - [Hide side bar](#hide-side-bar)
- [Configure](#configure)
  - [Keybindings](#keybindings)
  - [Permissions](#permissions)
  - [Change the leader key sequence](#change-the-leader-key-sequence)
  - [State options](#state-options)
- [Providers](#providers)
  - [OpenRouter](#openrouter)
  - [Google Vertex](#google-vertex)
- [Model Comparisons](#model-comparisons)
  - [Kimi](#kimi)

## Overview
OpenCode being open source has some really neat options:
* automatically copying text by just selecting it with the left mouse button

## Tips

### Variants
OpenCode uses the term `variants` controlled with `Ctrl+t` to indicate the level of effort the model
should use e.g. Claude Code's Sonnet defaults to `medium` but you can change it to `low`, `medium`,
`high` or `max`.

## Configure
Because `OpenCode` is open soure it follows XDG conventions and stores its configuration in
`~/.config/opencode`

### Change the leader key sequence
Edit `~/.config/opencode/tue.json`
```json
{
  "$schema": "https://opencode.ai/tui.json",
  "keybinds": {
    "leader": "ctrl+space"
  }
}
```

### State options
For whatever reason OpenCode has spread its configuration across a few files.
`~/.local/state/opencode/kv.json` is responsible for the following configuration values.

```json
{
  "sidebar": "auto",
  "thinking_visibility": false,
  "timestamps": "hide",
  "tool_details_visibility": true,
  "assistant_metadata_visibility": true,
  "scrollbar_visible": true,
  "header_visible": true,
  "animations_enabled": true,
  "generic_tool_output_visibility": false,
  "theme": "catppuccin-macchiato"
}
```

### Keybindings
[OpenCode keybindings docs](https://opencode.ai/docs/keybinds/)

| Function                              | Key sequence
|---------------------------------------|-------------------------
| Switch agent mode i.e. `plan/build`   | `tab`
| Toggle right hand sidebar             | `<leader>` then `b`
| Edit the prompt in vim                | `<leader>` then `e`

### Permissions
The `~/.config/opencode/opencode.json` main configuration file has a permission section for
configuring granular permissions with the ability to set `ask`, `deny` or `allow`

#### Permission keys
By default OpenCode has read access to the current directory but you can use the `external_directory`
to ensure that other paths are also readable or editable.

| Key                 | What it controls
|---------------------|---------------------------------
| read                | File reads (matches file path)
| edit                | All file writes/edits
| bash                | Shell commands (matches the command)
| webfetch            | URL fetches
| task                | Subagent launches
| external_directory  | Access outside the working directory
| doom_loop           | Same tool call repeated 3x identically
| glob, grep, list    | File search operations


#### Permission examples
Wildcards: `*` maches any characters, `?` matches one character

```json
{
  "permission": {
    "bash": {
      "*": "ask",
      "git *": "allow",
      "rm *": "deny"
    },
    "edit": {
      "*": "deny",
      "src/**/*.ts": "allow"
    }
  }
}
```

## Providers
OpenCode supports a plethera of different LLM providers.

### OpenRouter
Historically reliable free slots on OpenRouter have been available for DeepSeek/Qwen releases:
- DeepSeek usually keeps a :free variant of its latest V3.x/R1 line
- Qwen usually keeps a :free variant of a mid-size Qwen3.x model
- Meituan's LongCat and some MiniMax models have had aggressive free-tier pushes

#### Configure OpenRouter
To configure OpenCode with OpenRouter, you're essentially replacing OpenCode's bac

1. Create your API key in OpenRouter
   1. Navigate to the [OpenRouter dashboard](https://openrouter.ai/settings/keys)
   2. Click `Create API Key` and copy the key
2. Configure OpenCode
   1. Launch OpenCode and run `/connect`
   2. Search for and select `OpenRouter` 
   3. Enter the API key
3. Choose your model
   1. Search for `free`

### Google Vertex
OpenCode has the ability to connect to Google's Vertex AI models.

1. Ensure you have gcloud installed
   ```nix
   {
     environment.systemPackages = [
       pkgs.google-cloud-sdk
     ];
   }
   ```
2. Configure ADC (Application Default Credentials)
   ```bash
   $ gcloud auth login --update-adc
   ```
3. Configure environment variables for project and location
   ```bash
   $ export GOOGLE_CLOUD_PROJECT="your-project-id"
   $ export VERTEX_LOCATION="global"
   ```
4. I don't believe this is necessary, but you can default your gcloud config as well
   ```bash
   $ gcloud config set project $GOOGLE_CLOUD_PROJECT 
   $ gcloud auth application-default set-quota-project $GOOGLE_CLOUD_PROJECT
   ```
5. Start OpenCode and choose your Google Vertex model
   1. Run `/models`
   2. Choose any model under the `Vertex` sections

## Model Comparisons

### Kimi
1. Kimi K2.6 — the standout pick. Its distinguishing benchmark is agentic stability: 4,000+ tool calls sustained over a 13-hour uninterrupted session, plus 80.2% SWE-Bench Verified and 58.6% SWE-Bench Pro (best of the group). APK RE/recompile loops are exactly this kind of long, multi-step, tool-heavy workflow (apktool/jadx → edit → gradle/apktool build → zipalign → sign → retest, repeat on failures) — a model that degrades over long sessions will drift or forget earlier constraints partway through.

### Qwen
2. Qwen3-Coder(-Next) — close second. Efficient MoE (only ~3B active params), 256K native context, Apache 2.0, and it's the most battle-tested in the wild (most-downloaded coding model as of Jan 2026) — strong for the raw code-editing/smali-diffing part, slightly less proven on very long agentic sessions than Kimi K2.6.

### GLM
3. GLM-5.1/4.6 — good agentic front-end/dev preference (Code Arena Elo 1,530) but less specifically benchmarked for sustained tool-call marathons.

If you're building an actual pipeline (harness driving apktool/jadx/gradle), Kimi K2.6 is the better bet specifically because of tool-call endurance; if you want something leaner to self-host, Qwen3-Coder-Next.

Sources:
- Kimi K2.6 vs GLM 5.1 vs Qwen 3.6 Plus vs MiniMax M2.7: Which Open Source Model Wins for Coding in 2026 - Atlas Cloud Blog (https://www.atlascloud.ai/blog/guides/kimi-k2-6-vs-glm-5-1-vs-qwen-3-6-plus-vs-minimax-m2-7-coding-2026)
- Best Open-Source Coding Model 2026: Kimi K3 vs GLM-5.2 vs DeepSeek V4 vs Qwen3 | Morph (https://www.morphllm.com/best-open-source-coding-model-2026)
- Qwen3-Coder-Next Technical Report (https://arxiv.org/html/2603.00729v1)
- Kimmy K2.6 and Qwen 3.6: The Open-Source Models Closing the Frontier Gap | MindStudio (https://www.mindstudio.ai/blog/kimmy-k2-6-qwen-3-6-open-source-frontier-models)
