# Terminals <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](..)
- [Session](#session)
  - [Attach](#attach)
- [Key Bindings](#key-bindings)
  - [Session Management](#session-management)
  - [Window Management](#window-management)
  - [Pane Management](#pane-mangement)
  - [Copy Mode & Navigation](#copy-mode-navigation)
  - [Help](#help)
- [Troubleshooting](#troubleshooting)
  - [Garbled Starship](#garbled-starship)

### Linked pages
- [TMUX](tmux/README.md)

## Sesions

### Attach
```bash
$ tmux attach
```

## Key Bindings
First hit the trigger key combination `Ctrl+Space` then one of the actions below:

### Session Management
* `s` – List and switch between sessions.
* `$` – Rename the current session.
* `d` – Detach from the session.

### Window Management
* `c` – Create a new window.
* `,` – Rename the current window.
* `w` – List windows (interactive menu).
* `n` / p – Switch to the next/previous window.
* `0`-9 – Select window by number.

### Pane Management
* `%` – Split vertically (side-by-side).
* `"` – Split horizontally (top-and-bottom).
* `z` – Toggle zoom (maximize/restore) the current pane.
* `x` – Kill the current pane.
* `A`rrow Keys – Navigate between panes.

### Copy Mode & Navigation
* `[` – Enter copy mode (allows scrolling and searching).
* `]` – Paste from the tmux buffer.
* `q` – Exit copy mode or a list menu.

### Help
* `?` – List all current key bindings.
* `:` – Enter the tmux command prompt.

## Plugins
The tmux plugin ecosystem is built around TPM (Tmux Plugin Manager). Below are the most popular and
"essential" plugins for a modern setup in 2025.

### The Essentials
* [tpm](https://github.com/tmux-plugins/tpm): The foundation. It allows you to install and update
  all other plugins with Prefix + I.
* [tmux-sensible](https://github.com/tmux-plugins/tmux-sensible): A set of "standard" settings
  that everyone wants but aren't defaults (e.g., increasing scrollback history, fixing the ESC key
  delay in Vim).
* [tmux-yank](https://github.com/tmux-plugins/tmux-yank): Solves the "copy to system clipboard"
  headache across different operating systems.

### Session & Workflow Management
* [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect): Manually save and restore your
  tmux environment (sessions, windows, panes) so they survive a reboot.
* [tmux-continuum](https://github.com/tmux-plugins/tmux-continuum): Works with resurrect to
  automatically save your sessions every 15 minutes and restore them when tmux starts.
* [tmux-fzf](https://github.com/sainnhe/tmux-fzf): Uses the fzf fuzzy finder to manage sessions,
  windows, and panes via a searchable popup menu.

### Aesthetics & Themes
* [Catppuccin for Tmux](https://github.com/catppuccin/tmux): Currently the most popular theme. It
  is highly customizable, clean, and supports Nerd Font icons (perfect for your Starship setup).
* [Tokyo Night](https://github.com/fabioluciano/tmux-tokyo-night): A beautiful dark theme that
  pairs well with matching Neovim/VS Code themes.

### Advanced Productivity
* [extrakto](https://github.com/laktak/extrakto): Press Prefix + g to fuzzy-find any text (URLs,
  paths, git hashes) currently on your screen and "extract" it to your clipboard or editor.
* [tmux-sessionist](https://github.com/tmux-plugins/tmux-sessionist): Provides extra keybindings
  for even faster session manipulation (creating, switching, and joining sessions).
* [tmux-which-key](https://github.com/nucc/tmux-which-key): If you forget your keybindings, this
  brings up a popup (similar to Neovim's which-key) to show you what's available.

Example configuration snippet (~/.tmux.conf):

# List of plugins
```
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'catppuccin/tmux'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @plugin 'tmux-plugins/tmux-yank'
```

# Initialize TMUX plugin manager (keep this line at the very bottom)
run '~/.tmux/plugins/tpm/tpm'

## Troubleshooting

### Garbled Starship
The garbled Nerd Font characters inside tmux are almost always caused by a `locale/UTF-8` mismatch or a
hardcoded `$TERM` variable.

Here are the most likely fixes for your setup:

1. Force tmux to use UTF-8 - If your locale environment variables ($LANG or $LC_ALL) aren't
   explicitly set to a UTF-8 locale when tmux starts, tmux will fail to render multi-byte characters
   like Nerd Font glyphs, resulting in garbled text.
 * Quick Test: Launch or attach to your tmux session using the -u flag to force UTF-8 support:
     tmux -u attach
     # or
     tmux -u new
 * Permanent Fix: Ensure ``~/.bashrc`` exports a UTF-8 locale before tmux starts:
   ```bash
   export LANG="en_US.UTF-8"
   export LC_ALL="en_US.UTF-8"
   ```

2. Fix $TERM Inside tmux - don't hard code `$TERM`

     set -g default-terminal "tmux-256color"
     set -as terminal-overrides ",xterm-256color*:RGB"
     # or use: set -ag terminal-overrides ",xterm-256color:Tc"


Summary of steps to take:
 1. Check for hardcoded export TERM=... in your shell config and remove it.
 2. Ensure LANG="en_US.UTF-8" is exported.
 3. Reload your shell and try starting tmux with tmux -u.
