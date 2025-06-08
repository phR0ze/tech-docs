# EER Model <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

EER Models

### Quick links
- [.. up dir](..)
- [MySQL Workbench EER Modeling](#mysql-workbench-eer-modeling)
  - [Install on NixOS](#install-on-nixos)
  - [Create a new EER Diagram](#create-a-new-eer-diagram)

## MySQL Workbench EER Modeling

### Install on NixOS
MySQL Workbench can be easily installed with:
```nix
environment.systemPackages = [
  pkgs.mysql-workbench
];
```

Or simply run without installing with:
```bash
$ nix-shell -p mysql-workbench
$ cd ~/Projects/oneup/api
$ mysql-workbench assets/db-model.mwb
```

### Create a new EER Diagram
1. Launch MySQL Workbench
2. Navigate to `Model >Add Diagram`
3. Place a table and double click it to add properties in the bottom view

### Add a foreign key to a table
1. Double click the table to bring up the property view
2. Select the `Columns` tab and scroll to the bottom where an empty row is
3. Double click the empty row add the new value e.g. `user_id`
4. Set the `user_id` data type to `INT`
5. Switch to the `Foreign Keys` tab
6. Set the `Foreign Key Name` e.g. `fk_user`
7. Set the `Referenced Table` e.g. `mydb.users`
8. Set the `Foreign Key Columns` e.g. `user_id` and the `Referenced Column` e.g. `id`
