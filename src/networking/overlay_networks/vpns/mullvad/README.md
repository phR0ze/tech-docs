# Mullvad <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

As of 2025 the privacy and security communities online have positioned Mullvad as the defacto 
standard for safe VPN use i.e. regular no log audits, etc...

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Registration](#registration)
  * [Vouchers](#vouchers)
  * [Configuration](#configuration)

**References**
* [Mullvad demo](https://www.youtube.com/watch?v=Z9y29Wxo060)

## Overview

### Registration
Mullvad offers anonymous registration. They don't require your personal information to create an 
account. They generate an account number and that is it. You can then login and add time to your 
account and setup your VPN configuration.

1. Navigate to Mullvad's [account create page](https://mullvad.net/en/account/create)
2. Click the `GENERATE ACCOUNT NUMBER`
3. Save your account number

### Vouchers
Mullvad offers a voucher system in addition to their other payments options so you can gift VPN time 
or alternatively it can be used as an extra measure of privacy. Because the vouchers are scratch to 
reveal the code there isn't any way to tie the purchaser of the voucher with the Mullvad account that 
the voucher was applied to. The easiest way to get vouchers through Amazon.

1. Login to your new account
2. Click `Add time to your account`
3. Click the `Voucher` option
4. Scratch your Amazon voucher and enter the code

### Configuration
1. Install the Mullvad client
   ```nix
   services.mullvad-vpn.enable = true;
   environment.systemPackages = [
     pkgs.mullvad-vpn
   ];
   ```
2. Launch Mullvad for configuration
   1. Ensure the daemon is running: `sudo systemctl start mullvad-daemon`
      * Note: if you see a weird error an no options it likely because the daemon is not running
   2. Launch the GUI interface `mullvad-gui`
3. Login to your account through the app
   1. Paste in your account code and click forward
