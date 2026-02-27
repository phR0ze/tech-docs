# Terminal Emulators

### Quick links

## Alacritty
Written in Rust and modern

**Features**
* Way faster rendering for VIM scrolling
* Slick precise window sizing rather than based on font size like xfce4-terminal
* Increase/decrease font size with `Ctrl++`/`Ctrl+-`/`Ctrl+0` - increase font size
* Ability to click on links
* Copy and paste hot keys `Ctrl+Shift+c` and `Ctrl+Shift+v`
* No tab support
* Updates window on the fly when editing the configuration
* With a window compositor set you can control the opacity of the terminal

**Issues**
* Currently not rendering my starship prompt correctly

### Configuration
The config file is located at `~/.config/alacritty/alacritty.yml` and example can be found at 
`/usr/share/doc/alacritty/example/alacritty.yml`

Get font names with `fc-list | grep JetBrains`

```yaml
window:
  padding:
    x: 5
    y: y
  class:
    instance: Alacritty
    general: Alacritty
scrolling:
  history: 10000
  multiplier: 3
background_opacity: 1
font:
  normal:
    family: JetBrains Mono Nerd Font
    style: Medium
  bold:
    family: JetBrains Mono Nerd Font
    style: Bold
  italic:
    family: JetBrains Mono Nerd Font
    style: MediumItalic
  bold_italic:
    family: JetBrains Mono Nerd Font
    style: BoldItalic
  size: 7
draw_bold_text_with_bright_colors: true

colors:
  primary:
    background: "#11121D"
    foreground: "#a9b1db"
  normal:
    black: "#32344a"
    red: "#f7768e"
    green: "#9ece6a"
    yellow: "#e0af68"
    blue: "#7aa2f7"
    magenta: "#ad8ee6"
    cyan: "#449dab"
    white: "#787c99"
  bright:
    black: "#444b6a"
    red: "#ff7a93"
    green: "#b9f27c"
    yellow: "#ff9e64"
    blue: "#7da6ff"
    magenta: "#bb9af7"
    cyan: "#0db9d7"
    white: "#acb0d0"

cursor:
  style:
    shape: Block
    blinking: Off
    blink-interval: 750

selection:
  save_to_clipboard: false

live_config_reload: true

shell:
  program: /usr/bin/bash

key_bindings:
  - { key: Return, mods: Super|Shift, action: SpawnNewInstance }
```

### Quake style drop down support
Supposedly merged in with https://github.com/alacritty/alacritty/pull/2786

1. lkj
