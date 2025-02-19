# SMB <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [SMB Configuration](#smb-configuration)
  * [Enable SMB](#enable-smb)
  * [Mount manually on client](#mount-manually-on-client)

## SMB Configuration

### Enable SMB
Enabling and configuring SMB is super simple

1. Navigate to `Control Panel > File Services > SMB`
2. Check `Enable SMB service`
3. Set the `Workgroup` e.g. `skynet`

### Mount manually on client
Mounting a SMB share locally on a client system can be done with the following:

```bash
# Using raw credentials
$ sudo mount -t cifs //SERVER/SHARENAME /mnt/temp -o username=username,password=password,workgroup=workgroup,iocharset=utf8,uid=username,gid=group

# Using a credentials file
$ sudo mount -t cifs //SERVER/SHARENAME /mnt/temp -o credentials=/etc/smb/secrets/Backup,iocharset=utf8,uid=username,gid=group
```

### Mounting user home directory
In order to mount a user's home directory using their credentials you need to:

1. Configure SMB access
   1. Navigate to `User & Group >User` and click `Edit`
   2. Switch to the `Applications` tab check `Allow` for `SMB`
2. Finally you can mount it by simply referring to `//SERVER_IP/home`
   ```bash
   $ sudo mount -t cifs //SERVER/home /mnt/temp -o credentials=/etc/smb/secrets/home,iocharset=utf8,uid=username,gid=group
   ```
