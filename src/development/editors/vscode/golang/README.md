# Golang

### Quick links
* [Install Golang](#install-golang)
* [Config Golang](#config-golang)
* [Debug with dlv](#debug-with-dlv)

### Install Golang
Install golang dependencies;

```bash
$ sudo pacman -S go go-tools go-bindata delve
```

### Config Golang
1. Set go path in `~/.bashrc`:    
   `export GOPATH=~/Projects/go`  

2. Install Extensions:  
   a. Click the button on the left that looks like an extension icon  
   b. Install ***Go ms-vscode.go***  

3. Add Golang Server Configs:  
   [Go Language](https://github.com/microsoft/vscode-go#go-language-server)  
   Note: you have to open a go file before the Go extension is officially activated  
   a. Hit `Ctrl+Shift+p` and enter `Preferences: Open Settings (JSON)` and paste  
   ```bash
   "go.gopath": "~/Projects/go", // Set the GOPATH to use
   "go.formatTool": "goimports", // Use specific external format tool for go
   "go.useLanguageServer": true, // Use the new gopls language server
   "go.languageServerExperimentalFeatures": {
       "documentLink": false, // Just reference local code via imports not external github pages with popup
       //"signatureHelp": false,
       // "format": true,
       // "autoComplete": true,
       // "diagnostics": true,
       // "goToDefinition": true,
       // "hover": true,
       // "goToTypeDefinition": true,
       // "findReferences": true,
       // "goToImplementation": true
   },
   "[go]": {
       "editor.snippetSuggestions": "none",
       "editor.formatOnSave": true,
       "editor.codeActionsOnSave": {
           "source.organizeImports": true
       }
   },
   "gopls": {
       "usePlaceholders": false, // add parameter placeholders when completing a function
       "completionDocumentation": true // for documentation in completion items
   },
   "files.eol": "\n", // Gopls formatting only supports LF line endings
   ```

### Debug with dlv
The golang extension that we installed ealier will fire the first time you open a golang project and 
ask if you want to download the missing helper tools, say yes to all and it will install `dlv-dap` 
into your go path and keep it up to date i.e. `~/Projects/go/bin` for cyberlinux.

`dlv-dep` should just work out of the box once you have a config setup below.

**Create a new debug config, example replace `PACKAGE` below with your package**
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/cmd/PACKAGE",
            "env": { },
            "args": [
            ]
        }
    ]
}
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
