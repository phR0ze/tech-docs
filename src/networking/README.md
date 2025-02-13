# Networking <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)

### Linked pages
- [Remoting](remoting/README.md)
 
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
