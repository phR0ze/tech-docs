# DBeaver <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Install DBeaver](#install-dbeaver)
* [Configure DBeaver](#configure-dbeaver)
  * [First run configuration](#first-run-configuration)

## Install DBeaver
NixOS packages has it available in the older versions as `dbeaver` and in latest unstable as 
`dbeaver-bin`.

1. Make available in a dev shell
   ```bash
   $ nix-shell -p dbeaver
   ```

2. Install on your system
   ```nix
   environment.systemPackages = [
     pkgs.dbeaver
   ];
   ```

## Configure DBeaver

### First run configuration
1. First run configuration
   1. Launch the app with `dbeaver`
   2. Click `No` to skip creating a local db
   3. Double click`MySQL` from the database choice list 
3. Configure a new connection
   1. Set `Server Host` to `example.com`  
   2. Set `Username` to `root`  
   3. Set `Password` to `1234567`  
4. Configure `MySQL` driver should be a one time thing
   1. Click `Test Connection...` at the bottom  
   2. Click the `Download` button at the bottom when asked to install drivers
   3. Finally click `Finish`

