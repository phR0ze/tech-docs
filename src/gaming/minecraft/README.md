# Minecraft <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Multiplayer](#multiplayer)
* [Prism Launcher](#prism-launcher)
  * [Initial configuration](#initial-configuration)
  * [Mod Support](#mod-support)
* [Mods](#mods)

## Overview

### Multiplayer
Minecraft is better when you play with your friends. You have a few options:

1. Self host and port forward but that is dangerous
2. Pay a monthly subscription fee to have your own limited Realm
3. Pay for a minecraft server from a 3rd party minecraft hosting company
4. Use the `Essential Mod` to allow players to host a single player world and invite your friends

## Prism Launcher
The minecraft client is usually what most gamers will need. Only those hosting their own servers will 
need the server. There are a number of clients available. Some of the clients have the ability to run 
mods. Prism launcher is by far the best client out there. Its open-source, it doesn't have 
advertisements or spyware. It's just a nice client that does what its supposed to.

### Initial configuration
1. Launch `PrismLauncher`
2. Select Java version `21.x.x`
3. Click next to accept that and the `512 MiB` min RAM and `4096 MiB` max RAM
4. Simply click `Finish` when prompted to add a Microsoft account
5. Add a server instance
   1. Click `Add Instance`
   2. Select the appropriate version to match the server e.g. `1.19.4`
   3. Click `OK` to finish
6. Create an offline account
   1. Click `Acounts`
   2. Select `Manage Accounts...`
   3. Click `Add Offline` on the right
   4. Enter your chosen name without spaces e.g. `HamSlam`
   5. Click `OK` to finish
   6. Click `Close` to return to the main screen
7. Click `Launch` on the right to start
   * Note: first launch may take awhile while it downloads assets
8. Connect to the server
   1. Once launched click `Continue` and click `Multiplayer`
   2. Accept the warning and then click `Add Server` 
   3. Specify the name e.g. `Foobar`
   4. Enter the local IP address e.g. `192.168.10.3`
   5. Click `Done`
   6. Finally double click on the new server entry

### Install Essential Mod
Prism Launcher integrates directly with Modrinth and CurseForge and can download and update 
individual mods in addition to modpacks. Prism Launcher can manage modpacks easily, while keeping 
them clean. You no longer have to install and update modpacks manually.

1. Create a new modable instance
   1. Select your desired version e.g. `1.21.1` in the top split
   2. Select the latest `Fabric` mod loader in the bottom split
   3. Click `OK` to create the new instance
2. Add mods to instance
   1. Right click new instance and select `Edit...`
   2. Select `Mods` on left
   3. Click `Download mods` on right
   4. Select mod source `CurseForge`
   5. Select `Fabric API` and click `Select mod for download`
   6. Select `Essential Mod` and click `Select mod for download`
   7. Click `Review and confirm`

## Mods
Minecraft has a dedicated fan based that creates their own modded versions of Minecraft that expand 
the capabilities of Minecraft. Modded Minecraft is a version fo the game that can run user-created 
add ons, a.k.a. mods. Minecraft doesn't support this out of the box, so a special version is 
required. There are several modded Minecraft clients available, `Forge`, and `Fabric` are two of the 
most popular. 

**References**
* [Minecraft Mods Begineers Guide - The Gamer](https://www.thegamer.com/minecraft-mods-beginners-guide-tips-explained/)

Modded Minecraft doesn't always mirror the latest version. Developers usually settle on a milestone 
and develop their mods for that version. Often you'll have to use an older modded client to play. 

### Essential Multiplayer

### Essential mod
[Essential mod](https://www.curseforge.com/minecraft/mc-mods/essential-mod) provides the ability to 
host your own Minecraft world, invite your friends and play together in just a few clicks.

### Teletransportation accept (tpa)
[Teletransportation Accept (TPA)](https://www.curseforge.com/minecraft/mc-mods/teletransportation-accept-tpa)
adds the possibilty to request to another player to teleport to him!. It is internally a datapack 
that needs Fabric API to work on Fabric. It makes use of the newly added macros in functions in 
Minecraft 1.20.2

### Attributefix
Minecraft uses an attribute system to handle many entity properties like max health, movement speed, 
and attack damage. While this system is very flexible Mojang has artifically limited many of these 
properties. [AttributeFix](https://docs.darkhax.net/mods/attributefix/) addresses this issue by 
significantly raising these limits to a reasonable range.

### LifeSteal Fabric
[LifeSteal](https://www.curseforge.com/minecraft/mc-mods/lifesteal) is simple fabric mod based on the 
LifeSteal SMP. It adds a command that can be used to modify and get a players lost health.

### Lithium
[Lithium](https://www.curseforge.com/minecraft/mc-mods/lithium) is a general-purpose optimization mod 
for Minecraft, which works to improve a number of systems (game physics, mob AI, block ticking, etc) 
without changing any behavior. It works on both the client and server, and can be installed on 
servers without requiring clients to also have the mod. With the mod installed, you can expect to see 
more than a 50% improvement to tick times, resulting in a much faster game.

### SDM shop
[SDM Shop](https://www.curseforge.com/minecraft/mc-mods/sdm-shop) adds an in-game store with which 
you can build your economy without needing villiagers.

### Simple voice chat
[Simple voice chat](https://www.curseforge.com/minecraft/mc-mods/simple-voice-chat)

### Tiny mob farm remastered 
[Tiny farm remastered](https://www.curseforge.com/minecraft/mc-mods/tiny-mob-farm-remastered) adds 
different tiers of single-block size mob farms that generates mob loots over time. Different tiers of 
mob farms have different speed; the Wooden Mob Farm is the lowest tier and the slowest, then the 
Stone Mob Farm, etc.

### Waystones
[Waystones](https://www.curseforge.com/minecraft/mc-mods/waystones) adds waystone blocks that the 
player can return to once they've been activated, either through a Warp Scroll a rechargeable Warp 
Stone or by using an existing waystone to hop from one to the other.

### Combatlog
[Combatlog](https://www.curseforge.com/minecraft/mc-mods/combatlog) tries to prevent people from 
combat logging on a server, after a player is hit by another player, both of them has a configurable 
countdown assigned to them and if either of them leave before that count down is over they are killed 
and all of their items are dropped on the ground.

### Just Enough Items (Jei)
[JEI](https://www.curseforge.com/minecraft/mc-mods/jei) is an item and recipe viewing mod for 
Minecraft, built from the ground up for stability and performance.

<!-- 
vim: ts=2:sw=2:sts=2
-->
