# Terminal

## Colors
MacOS doesn't seem to recognize `alacrity` as a valid color terminal so in order to get a remote SSH 
session to show color when shelling into a MacOS system, you'll need to add the following to your 
`.bashrc` to enable color.

Seems to be MacOS's default TERM value
```
export TERM=xterm-256color
```
