# Dev Containers

### Quick links
* [Prerequsites](#prerequisites)

Scripting languages like Ruby and Python frequently roll out changes to the language and library 
pieces breaking a system install. To keep the system up to date with the madness is a full time job. 
The only way to develop sanely is to do so from a stable development environment and that is what the 
[Developing inside a container](https://code.visualstudio.com/docs/devcontainers/containers) 
extension and paradigm brings. It is the ability to have a stable well defined environment that won't 
change unless you want it to.

Workspace files are mounted from the local file system. Extensions are installed and run inside the 
container, where they have full access to the tools, platform, and file system. This means that you 
can seamlessly switch your entire development environment just by connecting to a different 
container.

![Dev inside container image](images/dev-env-inside-container.png)

This lets VS Code provide a local-quality debugging experience including full code completions, code 
navigation, and debugging regardless of where your tools and code are located.

1. You can use a container as your full-time development environment
2. You can attach to a running container to inspect it

## Prerequisites
1. You need VS Code installed
2. You need Docker installed

The `devcontainer.json` file in your project tells VS Code how to access or create a development 
container.

<!-- 
vim: ts=2:sw=2:sts=2
-->
