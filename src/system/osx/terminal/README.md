# Terminal <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)

## Colors
MacOS doesn't seem to recognize `alacrity` as a valid color terminal so in order to get a remote SSH 
session to show color when shelling into a MacOS system, you'll need to add the following to your 
`.bashrc` to enable color.

Seems to be MacOS's default TERM value
```
export TERM=xterm-256color
```
