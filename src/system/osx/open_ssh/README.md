# OpenSSH
MacOS comes with an OpenSSH server installed by default. You can enable it in 
`System Preferences >General >Sharing` section.

## Mobile Device Management workaround
Fleet systems will often have SSH disabled such that you can't ssh into your system. To work around 
this silly limitation you can simply run a dedicated copy of the OpenSSH server on a new port.

1. Create the file `~/.ssh/authorized_keys`
2. Add you Public Key to the file
3. Set the permissions to remove read access for other and all: `chmod go-r ~/.ssh/authorized_keys`
4. Create a custom sshd config file `sshd_config`
   ```
   Port 2222
   PubkeyAuthentication yes
   AuthorizedKeysFile	.ssh/authorized_keys
   UsePAM no
   ```
5. Run OpenSSH server in debug mode to test it
   ```bash
   $ sudo /usr/sbin/sshd -d -D -f sshd_config
   ```
6. Login from your remote system with the client
   ```bash
   $ ssh -p 2222 user@192.168.1.2
   ```
7. Run OpenSSH server in background accepting multiple connections
   ```bash
   $ sudo /usr/sbin/sshd -f sshd_config
   ```

## Troubleshooting

### Get list of listening services
* `-n` do not convert port numbers to port names
* `-P` do not resolve hostnames, show numerical addresses
* `-iTCP:22 -sTCP:LISTEN` show network files with TCP state LISTEN on port `22`

```bash
sudo lsof -nP -iTCP -sTCP:LISTEN
COMMAND   PID      USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
launchd     1      root    7u  IPv6 0xbd0d88da47a08781      0t0  TCP *:22 (LISTEN)
launchd     1      root    8u  IPv4 0x534b1bf3b1799733      0t0  TCP *:22 (LISTEN)
launchd     1      root   10u  IPv6 0xbd0d88da47a08781      0t0  TCP *:22 (LISTEN)
launchd     1      root   11u  IPv4 0x534b1bf3b1799733      0t0  TCP *:22 (LISTEN)
```

### Get your ip address
```bash
$ ipconfig getiflist
en4 en5 en6 en0 en8
$ ipconfig getifaddr en0
192.168.1.239
```
