# RuntimePath <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

The `runtimepath` is similar to the shell environment `PATH` value on Unix systems. Vim will search 
in these paths to locate and source many different files during startup. You can see the content of 
the runtimepath by running `:set runtimepath?`

**Source order**
1. `$XDG_CONFIG_HOME/nvim` or `~/.config/nvim` if not set
2. `$VIMRUNTIME`
3. `$XDG_CONFIG_HOME/nvim/after`

### Quick links
- [.. up dir](../README.md)
- [Vim Startup](#vim-startup)

## Vim Startup

### Summary of startup
1. Set the shell options
2. Process the arguments given to the cli and create any necessary buffers
3. Execute the value of the `$VIMINIT` variable
4. Source the users `init.lua`
5. Execute the value of the `$EXEINIT` variable
6. If the `exrc` is set find and load a vimrc
7. Source filetypes, filetype plugins, and indent plugins
8. Source syntax highlighting scripts
9. Source plugin scripts
10. Load runtime and plugin scripts from the `after` folder
11. Source the shada file
12. Execute the options given to Vim affecting the startup

### Profiling Vim's startup
* Add the `-V` flag to increase the verbosity.
* Run with `vim --startuptime <file>` to log started scripts and how long they took

### ftplugin directory
The subdirectory `ftplugin` for file type plugins allows you to load pieces of configuration each 
time you open a buffer with a specific filetype.

### autoload directory
The subdirectory `autoload` is also very useful. It lets you load custom functions when you call 
them, instead of loading them when Vim starts. As a result you can significantly speed up Vim's 
startup time by postponing configuration until you need it rather than at boot time.

### syntax directory
THe subdirectory `syntax` lets you create you own syntax files. They're used for code highlighting 
for example.

### colors directory
THe subdirectory `colors` is for highlight groups for color.

### ftdetect directory
THe subdirectory `ftdetect` is for file types of your own creation

### after directory
THe subdirectory `after` is a special one. You can think of it as another runtime path in the runtime 
path where you can add the same directories you saw above. Everything in the after directory will be 
loaded after everything else.

### Manually source files
You can manually source files with `:runtime <file>` e.g. to source all files from colors `:runtime 
colors/**/*.vim`
