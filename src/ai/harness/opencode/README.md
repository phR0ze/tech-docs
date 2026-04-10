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
  - [Google Vertex](#google-vertex)
  - [OpenRouter](#openrouter)

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

### OpenRouter
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
