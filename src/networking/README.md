# Networking
Documenting various networking technologies

### Quick links
* [iptables](#iptables)
* [Internet Archive](#internet-archive)
  * [wget bulk downloads](#wget-bulk-downloads)
* [Network Interfaces](#network-interfaces)
  * [Bind to NIC](#bind-to-nic)
  * [Configure Multiple IPs](#configure-multiple-ips)
* [SSH](#ssh)
  * [Port Forwarding](#ssh-port-forwarding)

# iptables
`netfilter` controls access to and from the network stack at the Linux kernel module level. The
primary command line tool for managing netfilter hooks has been iptables rulesets. However since the
syntax needed to invoke those rules is complicated various user friendly tols like `ufw` and
`firewalld` have been created to simplify this.

Pro tips:
* rules are order specific later rules taking priority
* manually applied rules with `sudo iptables` are applied immediately
* iptable rules don't persist by default
* `sudo iptables -S` - list out current iptables

## iptables reset
```bash
$ sudo systemctl restart iptables
```

## kiosk
To illustrate iptables let's imagine we want to setup a Kiosk for a local school. The intent is that
students would be able to use a browser in a limited way to complete school related work. The kiosks
would need access to `ixl.com`. To keep the example simple we'll ignore all other protocols and
simply focus on `HTTP` and `HTTPS` over ports `80` and `443` and only concern ourselves with outbound
connections ignoring inbound.

This simple set of rules says to drop all outbound traffic on port 80 or 443 except to `ixl.com`:
```bash
sudo iptables -A OUTPUT -p tcp -d ixl.com --dport 80 -j ACCEPT
sudo iptables -A OUTPUT -p tcp -d ixl.com --dport 443  -j ACCEPT
sudo iptables -A OUTPUT -p tcp --dport 80 -j DROP
sudo iptables -A OUTPUT -p tcp --dport 443 -j DROP
```

* `-A` - indicates were adding a rule
* `OUTPUT` - indicates the rule should be part of the output chain
* `-p tcp` - indicates that this rule will apply only to packets using the TCP protocol
* `-d ixl.com` - indicates the destination
* `-j ACCEPT` - indicates the action to take
* `--dport 80` - indicates requests bound for destination port 80

# Internet Archive

## Advanced searching
Often public domain media will show up in the internet archive and you can find it with the advanced 
search capabilities.

1. Navigate to [https://archive.org/advancedsearch.php](https://archive.org/advancedsearch.php)

## wget bulk downloads
[Internet Archive blog source](https://blog.archive.org/2012/04/26/downloading-in-bulk-using-wget/)

**wget options:**
* `-r` - recursive download
* `-H` - go to foreign hosts when recursive
* `-nc` - skip downloads that would overwrite existing files
* `-np` - don't ascend to the parent directory
* `-nH` - don't create host directories
* `--cut-dirs=1` - ignore 1 remote directory components
* `-A .mp4` - comma-separated list of accepted extensions
* `-e robots=off` - ignore robot exclusion directives
* `-l1` - set max recursion depth to 1
* `-i episodelist.csv` - csv of download links to process
* `-B 'r/http://archive.org/download'` - resolve links relative to this base URL

```bash
$ wget -r -H -nc -np -nH --cut-dirs=1 -A .mp4 -e robots=off -l1 -i episodelist.csv -B 'r/http://archive.org/download/'
```

# Network Interfaces

## Bind to NIC
Many Linux apps will by default bind to all available network interfaces. This is a nightmare for
end users that want control of which nics are used. To get around this many techniques have been
developed to replace code at runtime using LD_PRELOAD shims to inform the dynamic linker to first
load all libs into the process that you want then add some more.

```bash
# Compile code
wget http://daniel-lange.com/software/bind.c -O bindip.c
gcc -nostartfiles -fpic -shared bindip.c -o bindip.so -ldl -D_GNU_SOURCE

# Install binary
strip bindip.so
sudo cp bindip.so /usr/lib/

# Check existing binding of teamviewer
netstat -nl | grep 5938
tcp        0      0 0.0.0.0:5938            0.0.0.0:*               LISTEN     

# Edit teamviewer unit and add teh new start up line below
sudo systemctl stop teamviewerd
sudo sed -i -e 's|^\(PIDFile=.*\)|\1\nEnvironment=BIND_ADDR=10.33.234.133 LD_PRELOAD=/usr/lib/bindip.so|' /usr/lib/systemd/system/teamviewerd.service
sudo systemctl daemon-reload
sudo systemctl start teamviewerd

# Check resulting binding
netstat -nl | grep 5938
tcp        0      0 10.33.234.133:5938      0.0.0.0:*               LISTEN     
```

## Configure Multiple IPs
In Linux its easy to add more than one ip address to a given NIC. But if your service binds to all
NICs then your not going to get an open port, use [Bind to NIC](#bind-to-nic) to limit the service
to a specific IP address then create another to forward ports to.

```bash
# Add an address to your NIC
sudo tee -a /etc/systemd/network/20-dhcp.network <<EOL

[Address]
Address=192.168.0.1/24
[Address]
Address=192.168.0.2/24
EOL

# Restart networking for it to take affect
sudo systemctl restart systemd-networkd

# Check new IPs exist
ip a
# inet 192.168.0.1/24 brd 192.168.0.255 scope global enp0s25
# inet 192.168.0.2/24 brd 192.168.0.255 scope global enp0s25
```

# SSH

## Port Forwarding
Securely forwarding ports via ssh is simple just hard to remember.

```bash
# Forward local host:port to remote host:port using the ssh connection
# e.g. forwards local 192.168.0.1:5938 to remote 192.168.1.10:5938 via user@access-point.com
# 192.168.1.10 in this case is a host accesible from access-point.com

# ssh -L local_host:local_port:remote_host:remote_port user@access-point.com
ssh -L 192.168.0.1:5938:192.168.1.10:5938 -p 23 user@access-point.com

# Forward 5938 from access-point.com directly
ssh -L 192.168.0.1:5938:127.0.0.1:5938 -p 23 user@access-point.com

# Check resulting ports
netstat -nl | grep 5938
tcp        0      0 192.168.0.1:5938        0.0.0.0:*               LISTEN     
tcp        0      0 10.33.234.133:5938      0.0.0.0:*               LISTEN     
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
