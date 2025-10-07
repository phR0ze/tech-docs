# Word Processing <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

To turn Vim into an efficient word processor for writing, you need to configure specific settings and 
install plugins that provide features like distraction-free mode, spell checking, and easy text 
navigation

### Quick links
- [.. up dir](../README.md)

## Basic word processor settings
```
setlocal wrap               " Turn on soft line wrapping
setlocal textwidth=80       " Set maximum line length to 80 characters
setlocal formatoptions=t1   " Auto-format text; `t` wraps, `1` avoids breaking words
setlocal spell              " Enable Vim's built-in spell checker
setlocal spelllang=en_us    " Set spell-check language
setlocal nolist             " Don't show non-printable characters
setlocal linebreak          " Wrap lines at word boundaries instead of mid-word
```

### Create a function to enable settings
Creating a function that will enable word processing settings as needed is a convenient way to manage 
the desired settings in conjunction with standard development settings.

```
function! WordProcessorMode()
  setlocal formatoptions=t1
  setlocal textwidth=80
  setlocal wrap
  setlocal spell
  setlocal spelllang=en_us
  setlocal noexpandtab
  map j gj
  map k gk
  " Call Goyo for distraction-free mode
  Goyo
endfunction
command! WP call WordProcessorMode()
```
