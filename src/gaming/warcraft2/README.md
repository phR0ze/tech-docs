# Warcraft 2 <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Warcraft2 is a classic real time strategy game that Blizzard Entertainment made back in the 90s. This 
documentation is for the ***Battle.net Edition*** from ***GOG***.

Note: these directions should work regardless of your particular version of Linux once you get Wine 
installed that is.

### Quick links
* [.. up dir](..)
* [Install Warcraft 2](#install-warcraft-2)
  * [Install Wine](#install-wine)
  * [Install GOG binary](#install-gog-binary)
* [How to play](#how-to-play)
  * [Host LAN multi player game](#host-lan-multi-player-game)

## Overview
**References**
* [Lutris Script](https://lutris.net/games/install/12552/view)
* [Wine HQ guide](https://appdb.winehq.org/objectManager.php?sClass=version&iId=592)

## Install Warcraft 2

### Install Wine
see [Install Wine](../wine#install-wine)

### Install GOG binary
By default the GOG installer will install to `C:\GOG Games\Warcraft II BNE` which with our prefix
will be `~/.wine/prefixes/warcraft2/drive_c/GOG\ Games/Warcraft\ II\ BNE/`

1. Purchase Warcraft 2 from [GOG](https://www.gog.com)

2. Download using the `Download offline backup game installers` option

3. Create and configure new ***win32*** wine prefix
   ```bash
   mkdir -p ~/.wine/prefixes
   WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 winecfg
   ```

4. Set the target windows version:
   1. Navigate to the `Applications` tab
   2. Choose `Windows Version: >Windows 7` at the bottom

5. Set the default audio devices:
   1. Navigate to the `Audio` tab
   2. Select your audio devices. `System default` is fine for systems with a single audio output. 
      Otherwise choose your preferred device e.g. `USB Audio Device Analog Stero`
   3. Click the `Test Sound` button and ensure you hear sound
   4. Close Wine configuration by clicking `OK`

6. Install the game
   1. Launch the installer
      ```bash
      WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 wine ./setup_warcraft_ii_2.02_v4_\(28734\).exe
      ```
   2. Select Setup Language: `English` and click `OK`
   3. Check `Yes, I have read and accept EULA` then click `Install`
   4. Click `Exit`

7. Setup Direct X windowing:  
   1. Launch the direct x config tool  
      ```
      WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 wine ~/.wine/prefixes/warcraft2/drive_c/GOG\ Games/Warcraft\ II\ BNE/dxcfg.exe
      ```
   2. Set `Display mode> 1280x1024` with the fastest `Hz` refresh your display supports 
   3. Set `Presentation >Windowed`  
   4. Set `Enhancements >Anisotropic filtering > Enabled - AF 16x`    
   5. Set `Enhancements >Antialiasing > Enabled - MSAA 8x`    
   6. Click `Save` then `Exit`  

8. We need to use the `wsock32.dll` supplied by GOG for IPX to work. No copying or moving of the
   `wsock32.dll` is needed. Instead simply set the override to not use the builtin one and it will
   automatically find the correct one and work. 
   1. Launch wine config to override the dll  
      ```
      WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 winecfg
      ```
   2. Select the `Libraries` tab
   3. Enter `wsock32` into the `New override for library` drop down
   4. Click `Add` accept the warning and you'll see `wsock32(native,builtin)` in the `Existing Overrides`
   5. Click `Apply` and `OK`

9. Setup IPX Configuration:  
   1. Launch the IPX configuration binary sitting along side the game files:
      ```
      WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 wine ~/.wine/prefixes/warcraft2/drive_c/GOG\ Games/Warcraft\ II\ BNE/ipxconfig.exe
      ```
   2. Select your `Primary Interface` e.g. `enp1s0`  
   3. Select your `Network adapters` e.g. `enp1s0`  
   4. Ensure `Enable Interface` is checked  
   5. Ensure `Enable Windows 95 SO_BROADCAST bug` is checked  
   6. Ensure `Automatically create Windows Firewall exceptions` is checked  
   7. Click `Apply` then `OK`  

## How to play

### Single player game
1. Launch warcraft 2:
   ```
   WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 wine ~/.wine/prefixes/warcraft2/drive_c/GOG\ Games/Warcraft\ II\ BNE/Warcraft\ II\ BNE_dx.exe
   ```

2. Click through the intro

3. Navigate to `Single Player Game`


### Host LAN multi-player game
1. Launch warcraft 2:
   ```
   WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/warcraft2 wine ~/.wine/prefixes/warcraft2/drive_c/GOG\ Games/Warcraft\ II\ BNE/Warcraft\ II\ BNE_dx.exe
   ```
2. Click through the intro
3. Navigate to `Multi Player Game > Enhanced >IPX network`
4. Click `Connect`
5. Enter your desired username e.g. `foobar`
6. Click `Create Game`
7. Have other players follow same process then `Join Game`

<!-- 
vim: ts=2:sw=2:sts=2
-->
