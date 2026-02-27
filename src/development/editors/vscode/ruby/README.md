# Ruby

### Quick links
* [Config Ruby](#config-ruby)
* [Debug Ruby](#debug-ruby)

**References**
* [Visual Studio Code Ruby guide](https://code.visualstudio.com/docs/languages/ruby)

### Config Ruby
`Ruby LSP` is meant to replace Solargraph, rubocop, byebug and others

1. Launch `code`
2. Install Ruby version manager
   1. Download AUR package: `yay -Ga rbenv`
   2. Build: `makepkg -s`
   3. Install: `sudo pacman -U rbenv-1.2.0-1-any.pkg.tar.zst`
   4. Add `eval "$(rbenv init - bash)"` to your `~/.bashrc`
3. Install Ruby LSB extension 
   1. Navigate to extensions
   2. Install `Ruby LSP` from `Shopify`
   3. Press `Ctrl+,` to load Settings
   4. Search for Ruby and select `Ruby LSP` on the left
   5. Select `Ruby Version Manager` drop down and pick `rbenv`

vscode config:
```json
    "[ruby]": {
        "editor.defaultFormatter": "Shopify.ruby-lsp",
        "editor.formatOnSave": true,
        "editor.formatOnType": true,
        "editor.tabSize": 2,
        "editor.insertSpaces": true,
        "files.trimTrailingWhitespace": true,
        "files.insertFinalNewline": true,
        "files.trimFinalNewlines": true,
        "editor.rulers": [
            120
        ],
        "editor.semanticHighlighting.enabled": true
    },
    "rubyLsp.enabledFeatures": {
        "codeActions": true,
        "diagnostics": true,
        "documentHighlights": true,
        "documentLink": true,
        "documentSymbols": true,
        "foldingRanges": true,
        "formatting": true,
        "hover": true,
        "inlayHint": true,
        "onTypeFormatting": true,
        "selectionRanges": true,
        "semanticHighlighting": true,
        "completion": true,
        "codeLens": true,
        "definition": true,
        "workspaceSymbol": true,
        "references": true
    },
    "rubyLsp.pullDiagnosticsOn": "save",
    "rubyLsp.rubyVersionManager": "rbenv",
    "rubyLsp.formatter": "rubocop",
```

### Debug Ruby
https://dev.to/dnamsons/ruby-debugging-in-vscode-3bkj

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for rdebug-ide",
      "type": "Ruby",
      "request": "attach",
      "cwd": "${file}",
      "remoteHost": "127.0.0.1",
      "remotePort": "1234",
      "remoteWorkspaceRoot": "${file}"
    }
  ]
}
```
