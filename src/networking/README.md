# Network configuration

## Check for Open Ports

### Lsof (List Open Files)
Show a list of all open TCP ports
* `-n` do not convert port numbers to port names
* `-P` do not resolve hostnames, show numerical addresses
* `-iTCP -sTCP:LISTEN` show network files with TCP state LISTEN

**Example**
```bash
$ sudo lsof -nP -iTCP -sTCP:LISTEN
```
