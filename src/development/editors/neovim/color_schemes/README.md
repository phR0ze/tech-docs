# Color Schemes <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

NeoVIM supports switching color schemes

### Quick links
- [.. up dir](../README.md)
- [Configure color scheme](#configure-color-schemes)


## Configure color scheme
I've configured this in [nixos-config/options/apps/system/neovim/config/lua/init.lua](https://github.com/phR0ze/nixos-config/blob/main/options/apps/system/neovim/config/lua/init.lua

### Temp change
You can temporarily try out new color schemes with the color picker. Simply hit `<leader>fc` then
select the color scheme and press enter.

### Persist change
To persist the changes edit [nixos-config/options/apps/system/neovim/config/lua/init.lua](https://github.com/phR0ze/nixos-config/blob/main/options/apps/system/neovim/config/lua/init.lua
and set
```lua
vim.cmd.colorscheme("tokyonight-storm")
```
