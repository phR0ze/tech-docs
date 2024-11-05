# mycli <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Install mycli](#install-mycli)
* [Configure mycli](#configure-mycli)
  * [Configure connections](#configure-connections)
* [mycli operations](#mycli-operations)
  * [Connections](#connections)
    * [Connection string](#connection-string)
    * [Command line args](#commane-line-args)

## Install mycli
1. Make available in a dev shell
   ```bash
   $ nix-shell -p mycli
   ```

2. Install on your system
   ```nix
   environment.systemPackages = [
     pkgs.mycli
   ];
   ```

## Configure mycli

### Configure Connections
You can configure mycli to store any number of DB URL connection strings to assist in tracking your 
various DBs.

1. Edit your `~/.myclirc`
2. At the bottom under the **Data Source Names** `[alias_dsn]` section add your DSNs
   ```
   [alias_dsn]
   test = mysql://my_user:password@localhost:3306/test_db
   ```
3. There after you can connect to your pre-configured DSNs by name
   ```bash
   $ mycli -d test
   ```

## mycli operations

### Connections
Connections can be made using command line flags or via a connection string

#### Connection string
```bash
$ mycli mysql://USER:PASSWORD@URL:PORT
```

### Command line args
```bash
$ mycli -h HOST -P 3306 -uUSER -pPASSWORD
```

### Show Databases
```bash
$ show databases
$ exit
```

### Show current user permissions
```
show grants for current_user();
```
