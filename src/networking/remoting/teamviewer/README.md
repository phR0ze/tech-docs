# Teamviewer <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Teamviewer is a free for personal use and provides remote desktop solution that in my opinion 
provided one of the first decent out of box experiences out there. Mostly it just worked. My only 
really complaints are the constant nagging to remind you to play fair and their questionable business 
practices and the occastional kernel incompatibility that caused certain versions to not work.

**Summary**
* Pros
  * Just works most of the time
  * Has a mobile client as well that actually works
* Cons
  * Nagging reminders
  * Questionable business practices
  * Occastional kernel issues
* Result
  * I no longer use this solution

### Quick links
* [../](../README.md)
* [Overview](#overview)

## Overview
Typically I configure TV to only be accessible from my LAN and just use it locally or tunnel in over 
SSH if I'm remote.

1. Install Teamviewer
  ```bash
  sudo pacman -S teamviewer
  sudo systemctl enable teamviewerd
  sudo systemctl start teamviewerd
  ```
2. Autostart Teamviewer
  ```bash
  cp /usr/share/applications/teamviewer.desktop ~/.config/autostart
  ```
3. Configure Teamviewer  
  a. Start teamviewer: `teamviewer`  
  b. Click ***Accept License Agreement***  
  c. Navigate to ***Extras >Options***  
  d. Click ***General*** tab on left  
  e. Set ***Your Display Name***  
  f. Check the box ***Start Teamviwer with system***  
  g. Set drop down ***Incoming LAN connections*** to ***accept exclusively***  
  h. Click ***Security*** tab   
  i. Set ***Password*** and ***Confirm Password***  
  j. Leave ***Start Teamviewer with Windows*** checked and click ***OK*** then ***OK***  
  k. Click ***Advanced*** tab on left  
  l. Disable log files  
  m. Check ***Disable TeamViewer shutdown***  
  n. Click ***OK***  


