# Networking

### Interfaces, addresses, and routes

The command `ifconfig` is traditionally used to display information about
network interfaces. It has been replaced by tools from the iproute2 suite. The
command `ip addr` shows the configuration of network interfaces.

The command `ip link` displays information about interfaces themselves, and not
addresses. Statistics about interfaces activity is displayed with the `-s`
flag: `ip -s link`.

The commands `route` and `ip route` provide information about the routing
table, the paths to other network locations.

### Port information

#### `netstat`

If `netstat` is not installed on your system run `sudo yum install net-tools`
(RHEL/CentOS) or `sudo apt install net-tools` (Debian/Ubuntu).

To find which process or service is listening on a particular port (e.g. port
80):

```
$ netstat -ltnp | grep ':80'
```

We use the following options:

* `l` shows only listening pocket
* `t` displays TCP connections
* `n` shows numerical addresses
* `p` shows the process ID and name

The utility `ss` shows similar information:

```
$ ss -ltnp
```

You can also display your network interfaces with the flag -i`:

```
$ netstat -i
```

#### `lsof`

The command `lsof` can be used to check which port are currently listening:

```
$ sudo lsof -i -P -n | grep LISTEN
```

Here we use several options:

* `-i`: when not provided an Internet address, select the listing of
all Internet network files
* `-P`: inhibits the conversion of port names to port number for network files
* `-n`: inhibits the conversion of network numbers to host names for network files

### DNS utilities

#### `dig`

`dig` is a command line tool that provides DNS lookup and diagnostics. It
displays properties of hostnames or IP addresses, and its results are
relatively easy to parse.

You can display the A records associated with a hostname:

```
$ dig www.finance-scraping.com
```

The return value of `dig` contains several sections:

* Question section: a reaffirmation of the query made to the DNS
* Answer section: the results of the query.
* Authoritative section: the authoritative name servers that host the hostname's
  records.
* Additional section: any extra information provided by the resolver.

The `+short` is useful to make `dig`'s results less verbose.

```
$ dig www.finance-scraping.com +short
```

One can specify the *type* of record that are desired, as a keyword after the
hostname. For example, `ANY` will query for NS, SOA, MX, etc. If not argument
is provided, `dig` will query for A records.

```
$ dig www.finance-scraping ANY
```

One can choose the name server that should be queried.

```
dig +short myip.opendns.com @resolver1.opendns.com
```

The option `+trace` instructs `dig` to query the root name server and return
the results from every server in the delegation chain.

```
dig www.google.com +trace
```

#### `nslookup`, `host` and `systemd-resolve`

These two commands also provide DNS information about hosts and IP addresses by
querying name servers.

```
$ nslookup www.finance-scraping.com
```

or

```
$ host www.finance-scraping.com
```

or 

```
$ systemd-resolve www.finance-scraping.com
```

You can check the global and per-link DNS settings with `systemd-resolve`:

```
$ systemd-resolve --status
```

### Firewall

#### `iptables`

The tool `iptables` is a firewall included in many Linux distributions, that
allows to control rules (accept, deny, etc) to apply to traffic on several
chains (INPUT, FORWARD, OUTPUT by default).

You can list rules for a chain by running the following command (the *chain*
argument is optional):

```
$ sudo iptables -L <chain>
```

You can *flush* current rules from the firewall using the `-F` flag:

```
$ sudo iptables -F
```



`nmap -v -sT` perform a verbose TCP scan
`iperf` to check performance of network connection
`iptables` firewall
`iptables -L` to list policies
`iptables -A INPUT -p tcp --dport 21 -j DROP` to deny FTP input access
`tcpdump` like wireshark
