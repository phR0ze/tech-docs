# TMUX <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [Toubleshooting](#troubleshooting)

## Troubleshooting

### Copy paste issues
Hold `Shift` while clicking and dragging to bypass tmux and let your terminal emulator capture the
selection with `Ctrl+Shift+c`. Then when pasting back into tmux use `<Leader> ]` to paste.

### Make starship prompts work
TMUX by default doesn't seem to plya nice with UTF-8 unless the following environment variables are
set properly. 

Set this in your `~/.bashrc` to get automatically sourced
```bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### MacOS Keychain issues
Having to login to services that you are already logged into e.g. Claude Code CLI via SSH is a
classic macOS security behavior intersecting with how Tailscale's SSH works.

#### The Root Cause
Claude Code CLI and other services store authentication tokens in the macOS Keychain.

When you sit at your MacBook and log in, macOS uses your password to decrypt and unlock this keychain
automatically. However if you were to SSH in using cert authentication this process doesn't
automatically happen and thus the keychain remains locked for that SSH session.

#### Easy fix with TMUX
Simply fire up tmux locally on your MAC creating any sessions you'd like to work with. Then SSH in
and attach to those TMUX sessions to inherit the already unlocked macOS keychain.

