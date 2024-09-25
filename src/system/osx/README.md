# OSX
My highly opionionated configuration for OSX

* Updated
  * 2023.08.21 - Ventura 13.4.1

## Quick links
* [System settings](#system-settings)
  * [Change user password](#change-user-password)
  * [Dock bar](#dock-bar)
  * [Finder](#finder)
  * [Sudo](#sudo)
* [Keyboard](#keyboard)
  * [Set repeat speed ](#set-repeat-speed)
  * [Set function keys to work correctly](#set-function-keys-to-work-correctly)
* [Sound](#sound)
  * [Enable sound icon in control bar](#enable-sound-icon-in-control-bar)
  * [Change alert sound](#change-alert-sound)
* [Install apps](#install-apps)
  * [Install Chrome Extensions](#install-chrome-extensions)
  * [Brew](#brew)
  * [iterm2](#iterm2)
  * [Starship](#starship)
  * [Commander One](#commander-one)
  * [1Password](#1password)
  * [Spectacle](#spectacle)
  * [Slack](#slack)
  * [Caffeine](#caffeine)
  * [Barrier](#barrier)
* [Develop](#develop)
  * [Install NeoVim](#install-neovim)
  * [Install Git](#install-git)
  * [Install Go](#install-go)
* [Visual Studio Code](#visual-studio-code)
  * [Install vscode](#install-vscode)
  * [Configure vscode](#configure-vscode)
* [Keyboard Shortcuts](#keyboard-shortcuts)
* [Network](#network)
  * [DNS](#dns)
  * [Set CloudFlare DNS](#set-cloudflare-dns)
  * [Flush DNS Cache](#flush-dns-cache)
  * [LAN config](#lan-config)
* [Input](#input)
  * [Invert Scrolling](#invert-scrolling)
  * [Key Remapping](#key-remapping)
* [Troubleshooting](#troubleshooting)
  * [Not booting](#not-booting)

## System settings

### Change user password
1. Navigate to `System Settings >User & Groups`
2. Select the target user
3. Click `Change Password...`

### Dock bar
1. Long press on the launch bar other than the icons
2. Choose `Dock Settings...`
3. Changes size to way small
4. Set `Double-click a window's title bar to` -> `Minimize`
5. Set `Minimize windows into application icon` -> enabled
6. Set `Automatically hide and show the Dock` -> enabled

### Finder
1. Open the finder
2. Navigate to `Finder >Settings...`
3. Click the `Sidebar` tab
4. Check
   * Home directory
5. Uncheck
   * Airdrop

### Sudo
Configure passwordless access

1. Edit sudo list
   ```bash
   $ sudo visudo
   ```

2. Find the admin entry and change to
   ```
   %admin          ALL = (ALL) NOPASSWD: ALL
   ```

## Keyboard

### Set repeat speed
1. Navigate to `System Settings >Keyboard`
2. Change repeat rate to fastest
3. Change delay util repeat to shortest

Note best UI settings will still be ***crazy slow!!!***
Use shell command to set defaults higher but logout and back in is required
```
$ defaults write -g KeyRepeat -int 1
```

### Set function keys to work correctly
1. Navigate to `System Settings >Keyboard`
2. Under `Keyboard navigation` click the `Keyboard Shortcuts...` button
3. On the left select `Function Keys`
4. On the right toggle the switch

## Sound

### Enable sound icon in control bar
1. Navigate to `Settings >Control Center` 
2. Change Sound's option to `Always Show in Menu Bar`

### Change alert sound
1. Navigate to `Settings >Sound` 
2. Change the `Alert sound` to `Funky`

## Install apps

### Install Chrome Extensions
1. Navigate to the [Chrome Webstore](https://chrome.google.com/webstore)
2. Install `Okta Browser Plugin`
2. Install `1Password`
2. Install `Ublock Origin`

### Brew
OSX doesn't come with a package manager and all the software available in the MAC apps store is out
of date or lame. Thus enters [Homebrew](https://brew.sh/) which fills this need.

Installs to: `/opt/homebrew`

1. Install with:
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. Configure terminal for app management
   1. Navigate to `System Settings >Privacy & Security >App Managment`
   2. Flip the toggle to allow `iTerm` to manage apps

3. Install essential tools
   ```bash
   export PATH=/opt/homebrew/bin:$PATH
   brew install bash
   brew install bash-completion
   brew install binutils
   brew install diffutils
   brew install jq
   ```

4. Install python and link
   ```bash
   brew install python
   cd /opt/homebrew/bin
   ln -s python3 python
   ```

   Haven't installed yet for Ventura
   ```bash
   brew install ed --with-default-names
   brew install findutils --with-default-names
   brew install gawk
   brew install gnu-getopt --with-default-names
   brew install gnu-indent --with-default-names
   brew install gnu-sed --with-default-names
   brew install gnu-tar --with-default-names
   brew install gnu-which --with-default-names
   brew install gnutls
   brew install grep --with-default-names
   brew install make --with-default-names
   brew install openssh
   brew install ruby
   brew install watch
   brew install wget
   ``` 

5. Upgrade later with
   ```bash
   $ brew update && brew upgrade
   ```

### iterm2
Decent terminal with drop down F12 support

Note: you can run iterm2 by searching for `iterm`

1. Install `brew install iterm2`

2. Configure general settings, navigate to `Preferences >General`
   1. Under `Startup` set `Only Restore Hotkey Window`
   2. Under `Closing` uncheck all options

3. Configure HotKey Window, navigate to `Keys >Hotkey`:
   1. Click `Create a Dedicated Hotkey Window...`
   2. Set ***F12*** as the hot key and click ***OK***

3. Configure HotKey Window appearance, navigate to `Preferences >Profiles`:
   1. Select the new `Hotkey Window` profile then the `Window` tab
   4. Change `Transparency` to `10`
   4. Change `Style` to `Maximized`

4. Install Inconsolata-lgc font
   ```bash
   brew tap homebrew/cask-fonts
   brew install font-inconsolata-lgc-nerd-font
   ```

5. Edit iterm2's preferences
   1. Select target profiles and change `Text` tab
   2. Set font to `Inconsolata LGC Nerd Font`
   3. Set font size to `13`

6. Configure to autostart see [Autostart App](#autostart-app)

7. Configure bash as your default shell
   1. Edit `/etc/shells`
   2. Add in `/opt/homebrew/bin/bash`
   3. Drop in your `.bashrc` and `.dircolors`
   4. Create profile link `ln -s .bashrc .bash_profile`
   5. Source your new bashrc `source ~/.bashrc`

### Starship

1. Install starship
   ```bash
   $ brew install starship
   ```

2. Drop the starship initilization in your `.bashrc`
   ```
   eval "$(starship init bash)"
   ```

### Commander One
Commander One provides a file manager for MacOS which it doesn't have by default

1. Install Commander One
   ```bash
   $ brew install commander-one
   ```
2. Enable viewing hidden folders
   1. At the top of the screen to the right toggle the unmarked toggle switch
3. Navigatge to `Commander One >Settings...`
4. Select `General` tab
   1. Uncheck `Send anoymous usage statistics to Electronic Tream, Inc.`
5. Select `View` tab
   1. Check `Show full file name`

### 1Password
1. Install `brew install 1password`
2. Search for it with `Super + spacebar` and enter `1password`
3. Launch `1Password.app` and click `Open` 
4. Click `Sign In` then choose `Sign in with SSO`
5. Enter `https://<<BUSINES>>.1password.com/` as the `Sign-In Address`
6. Enter your `Email Address`
7. This will trigger a code check on your phone or other trusted device
8. Login and allow the new device and enter the code back in the laptop

### Spectacle
1. Install `brew install spectacle`
2. Allow Spectacle accessibilty access
   1. Navigate to `System Settings >Privacy & Security`
   2. Flip the toggle for Sepectcle
3. Configure the following
   1. Clear all options except the `Left Half` and `Right Half` options
   2. These are controled with `Option + Command + arrow` or `Alt+Super+arrow`
4. Configure to autostart see [Autostart App](#autostart-app)
5. Create our own system short cut for maximizing a window
   1. Navigate to `System Settings >Keyboard`
   2. Under `Keyboard navigation` click the `Keyboard Shortcuts...` button
   3. On left select `App Shortcuts` and click the little plus icon
   4. The title must be set to `Zoom`
   5. Set the shortcut to ⌥	⌘ ↑
6. Create our own system short cut for minimizing a window
   1. Navigate to `System Settings >Keyboard`
   2. Under `Keyboard navigation` click the `Keyboard Shortcuts...` button
   3. On left select `App Shortcuts` and click the little plus icon
   4. The title must be set to `Minimize`
   5. Set the shortcut to ⌥	⌘ ↓
   6. Repeat the same thing for `Minimise`; alternate spelling for some apps

### Slack
1. Install Slack
   1. Run: `brew install slack`
   2. Launch the app and click `Sign In to Slack` which will launch Chromium
   3. Enter email and click `Sign In with Email`
   4. Check email and enter code
   5. Select the `BUSINESS` workspace then click `Open`
   6. Chrome will popup a dialog asking to allow `BUSINESS.slack.com`... check yes and click `Open Slack.app`
2. Navigate to `Settings >Sidebar`
   1. Uncheck `Later`
   2. Uncheck `Slack Connect`
   2. Uncheck `Files`
   2. Uncheck `Canvases`
3. Navigate to `Settings >Themes`
   1. Choose `Dark` mode

### Caffeine
To keep your laptop from constantly falling asleep when reading documents

1. Install `brew install caffeine`
2. Run caffeine
3. Check `Automatically start Caffeine at login`
4. Check `Activate Caffeine at launch`
5. Uncheck `Show this message when starting Caffeine`
6. Set `Default duration` to be `Indefinetely`

## Barrier
Although it will crash occasionally the Rosetta virtualization does an ok job of running this despite 
not having an ARM version available see [issues on ARM](https://github.com/debauchee/barrier/issues/960)

* [plist help](https://www.launchd.info/)
* Reset barrier with `rm ~/Library/Preferences/com.github.Barrier.plist`

1. Install via brew
   ```bash
   $ brew install barrier
   ```
2. Manually test:
   1. Test in foreground: `barrierc -f -n client --disable-crypto 192.168.1.4`
   2. Cancel out when blocked from running
   3. Navigate to `System Settings >Privacy & Security`
   4. Scroll down to the `Security` section
   5. Note a section with your app and click `Open Anyway`
   6. Click `Open` on the popup
   7. Click the allow access button to navigate to settings
   8. Once in `System Settings >Privacy & Security >Accessibility` toggle for `Barrier`
2. Set your laptop to run barrier, create the file `~/bin/start_barrier`
   ```bash
   #!/opt/homebrew/bin/bash
   barrierc -n client --disable-crypto 192.168.1.4
   ```
3. Start barrier
   ```bash
   start_barrier
   ```

You might need to auth the app once for macOS to run it:
1. Control click on the `Barrier.app` in the finder and then choose `Open`
2. Click `Open System Preferences` and ensure `Barrier.app` is listed to control the computer

## Develop
First thing to do in OSX is install command line tools. Apparently you have to do this every time you
upgrade as well.

**Install OSX dev tools**
```bash
$ xcode-select --install
```
 
### Install NeoVim
1. Install neovim
   ```bash
   $ brew install neovim
   ```
2. Install neovim plugin manager
   ```bash
   $ curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
   ```
3. Install the config file
   * Create dir `mkdir -p ~/.config/nvim`
   * Copy from Google Drive to `~/.config/nvim/init.vim`
4. Install Plugins
   ```bash
   $ vim ~/.config/nvim/init.vim
   :PlugInstall
   :q
   ```

### Install Git
```bash
$ brew install git
```

### Install Go
Note: for this to work `/usr/local/bin` needs to be one of the first things in your `PATH`

1. Install Go
   ```bash
   $ brew install go
   ```
2. Now update your Go tools:
   1. Launch vscode it should automatically prompty you but if not do step b
   2. Hit `Ctrl+Shift+P` and type in `Go: Install/Update Tools`

## Visual Studio Code

### Install vscode
Install go and delve
```bash
$ brew install visual-studio-code
$ brew install delve
$ ln -s /usr/local/opt/delve/bin/dlv ~/Projects/go/bin/dlv
```

### Configure vscode
Settings are stored in `~/Library/Application Support/Code`

1. Install extensions
   1. Click the button on the left that looks like an extension icon
   2. Install General extensions:
      | Name                     | Identifier                            |
      | ------------------------ | ------------------------------------- |
      | Even Better TOML         | `tamasfe.even-better-toml`            |
      | Code Runner              | `formulahendry.code-runner`           |
      | Color Picker             | `anseki.vscode-color`                 |
      | Copilot                  | `GitHub.copilot`                      |
      | Copilot Chat             | `GitHub.copilot-chat`                 |
      | Dev Containers           | `ms-vscode-remote.remote-containers`  |
      | Github Markdown Preview  | `bierner.github-markdown-preview`     |
      | Github Pull Request      | `GitHub.vscode-pull-request-github`   |
      | Vim                      | `vscodevim.vim`                       |
      | VSCode Great Icons       | `emmanuelbeziat.vscode-great-icons`   |

   3. Install Golang extensions
      | Name                     | Identifier                            |
      | ------------------------ | ------------------------------------- |
      | Go                       | `golang.go`                           |

   4. Install Ruby extensions
      | Name                     | Identifier                            |
      | ------------------------ | ------------------------------------- |
      | Ruby                     | `Shopify.ruby-extensions-pack`        |
      | Rubo Cop                 | `rubocop.vscode-rubocop`              |

   6. Install Rust extensions
      | Name                     | Identifier                            |
      | ------------------------ | ------------------------------------- |
      | rust-analyzer            | `rust-lang.rust-analyzer`             |
      | CodeLLDB                 | `vadimcn.vscode-lldb`                 |
      | Crates                   | `serayuzgur.crates`                   |

2. Configure keyboard shortcuts
   1. Navigate to `Code >Settings...>Keyboard Shortcuts`
      | Name                                  | Key sequence      |
      | ------------------------------------- | ----------------- |
      | `New Window`                          | `Ctrl+Shift+n`    |
      | `Close Window`                        | `Ctrl+w`          |
      | `Copy`                                | `Ctrl+c`          |
      | `Cut`                                 | `Ctrl+x`          |
      | `File: Open File`                     | `Ctrl+o`          |
      | `File: Open Folder`                   | `Ctrl+k, Ctrl+o`  |
      | `File: Save`                          | `Ctrl+s`          |
      | `workbench.action.openSettings`       | `Ctrl+,`          |
      | `workbench.action.files.saveAll`      | `Ctrl+Shift+s,`   |
      | `workbench.action.closeActiveEditor`  | `Ctrl+w`          |
      | `settings.action.search`              | `Ctrl+f`          |

2. Configure fonts
   1. Set the font to: `InconsolataLGC Nerd Font Mono`

## Keyboard Shortcuts
| Symbol    | Key         | Symbol    | Key        |
| --------- | ----------- | --------- | ---------- |
| ⌘         | Command Key |  ←        | Left Key   |
| ⌃         | Control Key |  →        | Right Key  |
| ⌥	        | Option Key  |  ↑	      | Up Key     |
| ⇧         | Shift Key   |  ↓        | Down Key   |

| General | Description |
| --------- | ----------- |
| ⌘ ,       | Open preferenes for any app |
| ⌘ r       | Refresh the current window |
| ⌘ w       | Close the current window |

| Windowing | Description |
| --------- | ----------- |
| ⌥	⌘ c     | Center window on screen |
| ⌥	⌘ ←     | Left Half of screen |
| ⌥	⌘ →     | Right Half of screen |
| ⌥	⌘ ↑     | Maximize window |
| ⌘ m       | Minimize window |
| ⌘ tab     | The press up arrow |

| Chromium | Description |
| --------- | ----------- |
| ⌘ ←       | Navigate back |

## Network

### DNS
```bash
# List DNS entries, look at thelast section 'DNS configuration (for scoped queries)'
scutil --dns

# Test DNS resolution path
nslookup google.com
Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
Name:	google.com
Address: 172.217.1.206
```

### Set CloudFlare DNS
By setting the CloudFlare DNS entry explicitly rather than using DHCP DNS configuration from router
the DNS resolution issues I had dissappeared.

1. Open ***System Preferences/Network***
2. Click the ***Advanced...*** button button right
3. Select the ***DNS*** tab
4. Set ***1.1.1.1***

### Flush DNS Cache
Some times it is necessary to clear out the DNS cache of stale or incorrect DNS issues
```bash
sudo killall -HUP mDNSResponder; sudo killall mDNSResponderHelper;sudo dscacheutil -flushcache
```

### LAN Config
A cheap USB C adapter will get you LAN support which is a lot faster and more reliable than WiFi

1. Plugin your lan cable/adapter
2. Open ***Finder >System Preferences***
3. Open Network
4. Choose ***Belkin...B-C LAN*** and configure
5. Turn off wifi

## Troubleshooting

### Safe Boot
Sometimes you might get an app that is causing problems that starts on boot

1. Power down
2. Press power button to restart
3. When you see the apple or hear the boot sound hold down `Shift`
4. Login when the prompt appears and it then will appear to start like normal but then load `Safe Boot`
   and prompt for your user name and password
5. It booted with most things disabled
6. Went into `Preferences >Users & Groups` removed all startup items

### App Cleaner & Uninstaller
Alternately, because the system could partially function when in `Safe Boot` I surmised that a 3rd 
party app of some kind was affecting it. So I installed `App Cleaner & Uninstaller` and uninstalled 
***EVERYTHING*** and all autostart options and W00T it is booting again.

### Super dead
I ran into a major issue when upgrading to Big Sur. My system wouldn't boot after.

#### Recovery Mode
1. Choose Restart then hold down `Command + R` until after blinks Logo shows up
2. Let it load and click `Next` and login
3. Choose `Disk Utility` and `Continue`
4. Select `Macintosh HD` and then `First Aid`
5. Select `Macintosh HD Data` and then `First Aid`

#### Reinstall macOS
1. Hit power then hold down `Command + R` until Logo shows up and let go
2. It will then boot into recovery mode
3. Select your user and click `Next` and login
4. Select `Reinstall macOS Big Sur` you'll need to connect your laptop to WiFi
