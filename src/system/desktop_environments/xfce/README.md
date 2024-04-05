# XFCE

### Quick links
* [xfconf](xfconf/README.md)
* [xfce4-appearance](#xfce4-appearance)
* [xfce4-clipman](#xfce4-clipman)
* [xfce4-display-settings](#xfce4-display-settings)
* [xfce4-panel](#xfce4-panel)
* [xfce4-power-manager](#xfce4-power-manager)
  * [Toggle presentationi mode](#toggle-presentation-mode)
* [xfce4-terminal](#xfce4-terminal)

**References**

## xfce4-appearance

## xfce4-clipman

## xfce4-display-settings

## xfce4-panel
Features application launchers, panel menus, a workspace switcher and more.

**References**
* [Xfce4 Panel source](uhttps://gitlab.xfce.org/xfce/xfce4-panel)
* [Xfce4 Panel docs](https://docs.xfce.org/xfce/xfce4-panel/start)
* [xfce4-panel blog post](https://alexxcons.github.io/blogpost_8.html)
  * Notice that datetime and clock plugins were merged
  * Shows all possible layouts
* [Dev how to](https://wiki.xfce.org/dev/howto/panel_plugins)

### xfce4-panel plugins
XFCE panel supports plugins. It comes with some basic ones out of the box called ***internal***

## xfce4-power-manager

### Toggle presentation mode
```bash
$ xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -T
```

## xfce4-terminal

<!-- 
vim: ts=2:sw=2:sts=2
-->
