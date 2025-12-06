# File Management <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

File managment is another nice to have plugin option for Neovim. There have been a number of nice 
additions in this space as well. My goto has been `oil` and my new faviorite is `snaks explorer`.

### Quick links
- [.. up dir](../README.md)
- [Snacks Explorer](#snacks-explorer)
  - [Explorer moving or copying](#explorer-moving-or-copying)
  - [Explorer configuration](#explorer-configuration)
- [Oil](#oil)
  - [Oil configuration](#oil-configuration)

## Snacks Explorer
Snacks provides a new file picker that doubles as an file manager.

**Pros:**
* Tree like view
* Current selection row highlighted
* Open files shown with an indicator icon

**Cons:**
* Tree like structure
* No ability to move up in directory

### Explorer keys

* `<leader>/` grep current directory
* `.` set current directory as CWD
* `<Tab>` select files with
* `a` add a new file or directory if you end it in `/`
* `<BS>` go up one directory
* `c` copies by prompting for a new name
* `d` delete the current file with prompt
* `h` close directory
* `H` toggle hidden files
* `i` or `/` to focuse the input search box for filtering
* `I` or `<a-i>` toggle ignored files from .gitignore
* `l` or `<CR>` open directory
* `m` move the file if selected else falls back on renaming
* `o` open file with system application
* `P` toggle preview window
* `r` rename the current file
* `u` update/refresh the view
* `y` yank file
* `Z` close all directories

### Yank and Paste alternative
1. Select files with `<Tab>`
2. `y` to yank file or files 
3. Navigate to target directory
4. Press `p` to paste copies from register

### Explorer moving or copying
1. Select files with `<Tab>`
2. Navigate to the destination directory
3. Execute the operation:
   - Press `m` to move the selected files to the current directory
   - Press `c` to copy the selected files to the current directory

### Explorer configuration
I made a number of changes to the default to make it behave and look more like Oil did:

```lua
after = function()
  require("snacks").setup({                       -- Lua module path
    explorer = {
      replace_netrw = true,                       -- open snacks explorer instead
      trash = true,                               -- use the system trash when deleting files
    },
    picker = {
      sources = {
        explorer = {                              -- explorer picker configuration
          jump = { close = true },                -- close the explorer picker after opening a file
          layout = {
            preset = "dropdown",                  -- use this layout for your picker
            preview = false,                      -- don't show the preview window by default
          },
        },
      },
      layout = {
        preset = "dropdown",                      -- drop down file picker only without preview
        cycle = false,                            -- don't cycle to the begining after hitting the end
      },
      layouts = {
        dropdown = {                              -- Custom picker based on built in vscode preset
          hidden = { "preview" },                 -- don't want the preview
          layout = {
            backdrop = true,                      -- dims the backround making picker pop out
            row = 1,                              -- draw the picker this many rows from the top of the screen
            width = 0.8,                          -- percentage of screen width to use for picker
            min_width = 80,                       -- minimum number of columns to make the width of the picker
            max_width = 120,                      -- maximum number of columns to make the width of the picker
            height = 0.6,                         -- percentage of the screen height to user for picker
            border = false,                       -- don't draw the border as it collides with the search box
            box = "vertical",
            { win = "input", height = 1, border = true, title = "{title} {live} {flags}", title_pos = "center" },
            { win = "list", border = "hpad" },
            { win = "preview", title = "{preview}", border = true },
          },
        },
      },
    },
  })
end,
```

## Oil
Oil gives a fantastic experience and I may stay with it as the file manipulation is pretty intuitive.

### Oil configuration
My custom Oil configuration with layout fixes to:
* keep Oil responsive to window size changes
* move Oil to the top of the screen
* adjust the width and height and center it

```lua
return {
  -- File manager for Neovim
  -- depends on snacks which is in turn dependent on mini.icons
  "oil.nvim",
  cmd = "Oil",
  before = function()
    require("lz.n").trigger_load("snacks.nvim")
  end,
  after = function()
    require("oil").setup({
      default_file_explorer = true,               -- replace the default file explorer
      skip_confirm_for_simple_edits = true,       -- skip asking for confirmation
      delete_to_trash = true,                     -- where is the trash?
      columns = {
        "permissions",                            -- display file permissions
        "size",                                   -- display file size
        "icon",                                   -- display file name
      },
      view_options = {
        show_hidden = true,                       -- show dot prefixed files
      },
      float = {
        -- padding = 0,                           -- vertical and horizontal are the same which is aweful
        max_width = 0.8,                          -- max width for the floating window relative to the parent
        max_height = 0.6,                         -- max height for the floating window relative to the parent
        override = function(cfg)
          cfg["row"] = 1                          -- override row offset to 1 to pull floating window to top
          return cfg
        end,
      },
      keymaps = {
        ["<ESC>"] = "actions.close",              -- ESC to close
      },
    })
  end,
  keys = {
    { "-", "<cmd>Oil --float<cr>", desc = "Open oil" },   -- '-' to launch
  },
}
```
