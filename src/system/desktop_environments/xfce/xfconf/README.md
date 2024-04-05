# xfconf configuration
Xfconf is a hierarchical (tree-like) configuration system where the immediate child nodes of the root 
are called `channels`. All settings beneath the channel nodes are called `properties`.

* Valid channel and property name are composed of the ASCII US-English characters:
  * A-Z, a-z, 0-9, dash `-`, and underscore `_`.
* `<` and `>` are also allowed in property names but not channel names
* Property names are referenced by their full path under the channel
* Both property and channel names are case-sensistive.

**Examples:**
* Channel: `ExampleApp`, property: `/main/history-window/last-accessed`

**References**
* [xfconf gitlab source](https://gitlab.xfce.org/xfce/xfconf)
* [xfce docs - xconf](https://docs.xfce.org/xfce/xfconf/start)

### Quick links
* [Accessing configuration](#accessing-configuration)

## Accessing configuration
Xfconf stores all its configuration in xml files on disk at 
`~/.config/xfce4/xfconf/xfce-perchannel-xml`. There is a run-time service `xfconfd` that serves up 
requests from the `xfconf` cli. Additionally, and more dangerous, but can be managed outside xfce's 
tools you can just maniuplate the xml directly once `xfconfd` is not running.

### Manual xml change steps
1. Kill the Xfconfd service
   ```bash
   $ pkill xfconfd
   ```
2. Make the desired changes in `~/.config/xfce4/xfconf/xfce-perchannel-xml`

3. Logout and back in

## xfce4-panel plugins
XFCE panel supports plugins. It comes with some basic ones out of the box called ***internal***

### xfce4-power-manager
Configuration is stored in `~/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-power-manager.xml`
