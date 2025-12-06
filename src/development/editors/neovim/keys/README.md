# Keys <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

My keybindings for Neovim

### Quick links
- [.. up dir](../README.md)
- [Bufferline](#bufferline)
- [Persistence](#persistence)
- [Snacks](#snacks)
  - [Snacks Explorer](#snacks-explorer)
  - [Snacks Picker Window Navigation](#snacks-picker-window-navigation)
  - [Snacks Terminal](#snacks-terminal)

## Bufferline

| Keys          | Context   | Description
| ------------- | --------- | ----------------------------------------------------------------------
| `<leader>bl`  | `n`       | Delete buffers to the left
| `<leader>bp`  | `n`       | Toggle buffer pin
| `<leader>bP`  | `n`       | Delete non-pinned buffers
| `<leader>br`  | `n`       | Delete buffers to the right
| `[b`          | `n`       | Prev buffer
| `]b`          | `n`       | Next buffer
| `[B`          | `n`       | Move buffer prev
| `]B`          | `n`       | Move buffer next

## Persistence

| Keys          | Context   | Description
| ------------- | --------- | ----------------------------------------------------------------------
| `<leader>qs`  | `n`       | Restore session
| `<leader>qS`  | `n`       | Select session
| `<leader>ql`  | `n`       | Restore last session
| `<leader>qd`  | `n`       | Don't save current session

## Snacks

| Keys          | Context   | Description
| ------------- | --------- | ----------------------------------------------------------------------
| `<C-q>`       | `n`       | Close the current buffer or Neovim if the buffer is the last one
| `<leader>t`   | `n`       | Terminal toggle `<Esc><Esc>` puts terminal in mode where`<leader>t` can close it
| `<leader>fb`  | `n`       | Find buffers from buffer list
| `<leader>fg`  | `n`       | Find grep through files
| `<leader>fh`  | `n`       | Find help pages
| `<leader>ff`  | `n`       | Find files in current working directory
| `<leader>fk`  | `n`       | Find key maps
| `<leader>fp`  | `n`       | Find picker from picker list
| `<leader>fr`  | `n`       | Find recent files

### Snacks Explorer

| Keys          | Context   | Description
| ------------- | --------- | ----------------------------------------------------------------------
| `<leader>/`   | `list`    | Grep the selected directory
| `.`           | `list`    | Set current directory as CWD
| `<BS>`        | `list`    | Navigate up on directory
| `<Tab>`       | `list`    | Select files for operations
| `a`           | `list`    | Add a new file or directory if you end it in `/`
| `c`           | `list`    | Copies by prompting for a new name
| `d`           | `list`    | Delete the current file after prompting for confirmation
| `h`           | `list`    | Close directory in the explorer tree
| `H`           | `list`    | Toggle hidden files
| `i`, `/`      | `list`    | Navigate back to `input` window
| `I`, `<a-i>`  | `list`    | Toggle ignored files from `.gitignore`
| `l`, `<CR>`   | `list`    | Open file or directory
| `m`           | `list`    | Move the file if selected else falls back on renaming
| `o`           | `list`    | Open file with system application
| `p`           | `list`    | Paste the file that was yanked for copy operations
| `r`           | `list`    | Rename the current file
| `u`           | `list`    | Update/refresh the view
| `y`           | `list`    | Yank file into a register for file operations
| `Y`           | `list`    | Yank parts of the file path via a prompt selection
| `Z`           | `list`    | Close all directories

```
["<c-c>"] = "tcd",
["<c-t>"] = "terminal",
["Z"] = "explorer_close_all",
["]g"] = "explorer_git_next",
["[g"] = "explorer_git_prev",
["]d"] = "explorer_diagnostic_next",
["[d"] = "explorer_diagnostic_prev",
["]w"] = "explorer_warn_next",
["[w"] = "explorer_warn_prev",
["]e"] = "explorer_error_next",
["[e"] = "explorer_error_prev",
```

### Snacks Picker Window Navigation

| Keys          | Context   | Description
| ------------- | --------- | ----------------------------------------------------------------------
| `<c-i>`       | `i`, `n`  | Change focus to the `input` window
| `<c-l>`       | `i`, `n`  | Change focus to the `list` window
| `<c-p>`       | `i`, `n`  | Change focus to the `preview` window

### Snacks Terminal

| Keys          | Context   | Description
| ------------- | --------- | ----------------------------------------------------------------------
| `<leader>t`   | `n`       | Change focus to the `input` window
