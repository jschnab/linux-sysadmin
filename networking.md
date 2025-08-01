# Networking

### Interfaces, addresses, and routes

The command `ifconfig` is traditionally used to display information about
network interfaces. It has been replaced by tools from the iproute2 suite. The
command `ip addr` shows the configuration of network interfaces.

The command `ip link` displays information about interfaces themselves, and not
addresses. Statistics about interfaces activity is displayed with the `-s`
flag: 

```
$ ip -s link
```

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

* `l` shows only listening sockets
* `t` displays TCP connections
* `n` shows numerical addresses
* `p` shows the process ID and name

If you dot not see the process ID of a connection, it may be because your user
does not have enough privileges, use `sudo`.

The utility `ss` now replaces `netstat` and shows similar information:

```
$ ss -ltnp
```

You can also display your network interfaces with the flag `-i`:

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

The `+short` is useful to make `dig` results less verbose.

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

These commands also provide DNS information about hosts and IP addresses by
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
chains (INPUT, FORWARD, OUTPUT for the `filter` routing table, which is the
default one).

##### Generalities

###### Command arguments

* `-t`: table (if not specified, operates on `filter` table)
* `-A`: append rule to specified chain
* `-i`: match packets from specified network interface
* `-p`: match packets that use the specified protocol
* `--dport`: match packets that use the specified port
* `-s`: match packets coming from the specified source IP (can use mask)
* `-j`: jump to specified target
* `--to-destination`: works with `DNAT` target, changes destination address to the one specified

###### Hook points

Hook points are locations in the packet processing path that can be modified by
the firewall. Each hook point has a chain of rules.

They are named:
* `PREROUTING`: just as packets arrive from a network interface
* `INPUT`: just before they are delivered to a local process
* `FORWARD`: coming from one interface and going right back out another
* `POSTROUTING`: just before they leave a network interface (not associated
    with the `filter` table)
* `OUTPUT`: just after they are generated by a local process

###### Chains

Chains are associated with hook points and define packet processing rules.

A chain's policy determines the fate of packet that reach the end of the chain
without being captured.

`ACCEPT` and `DROP` can be used as the policy for a built-in chain.

`RETURN` is present by default on user-defined chains, and this cannot be
changed.

###### Tables

`iptables` comes with three built-in tables:
1. `filter`: Set policies for traffic allowed in an out. Has `FORWARD`,
   `INPUT`, and `OUTPUT` chains.
2. `mangle`: Specialized packet operation (e.g. modify IP options). Has all
   built-in chains.
3. `nat`: Used for network address translation (uses `OUTPUT`, `POSTROUTING`,
   and `PREROUTING`).

###### Packet flow

Packets travers chains, are presented to chains' rules one at a time in order.
If the packet does not match the rule criteria, it moves to the next rule in
the chain. If a packet reaches the end of the chain without matching, the
chain's policy is applied.

###### Targets

Builtin-targets:

* `ACCEPT`: Packet stops traversing the current chain and goes to the next
  stage of processing.
* `DROP`: Discontinue processing the packet (do not check it against other
    rules, chains, or tables).
* `QUEUE`: Send the packet to userspace (see libipq manpage).
* `RETURN`: From a rule in a user-defined chain, discontinue processing this
    chain and resume traversing the calling chain at the rule following the one
    that had this chain as its target (like a function call return). From a
    rule in a built-in chain, discontinue processing the packet and apply the
    chain's policy to it.

#### `iptables` example commands

You can list rules for a chain by running the following command (the *chain*
argument is optional):

```
$ sudo iptables -L <chain>
```

Another way to print rules is to use the `-S` flat, which gives a bit more
information about rules (like devices):

```
$ sudo iptables -S <chain>
```

You can *flush* current rules from the firewall using the `-F` flag:

```
$ sudo iptables -F <chain>
```

Let's go through the simple task of configuring web and SSH access.
When building firewall policies, it is useful to add a rule that will keep
existing connections open.

```
$ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

We are using the `conntrack` module that provides extensions to the core
functionality of `iptables`, and the `ctstate` command of this module to
match packets based on how they are related to packets we have already seen.
The `-A INPUT` will *append* the rule to the end of the chain. The `j ACCEPT`
(jump) tells what to do with the packets that match.

Now let's add two rules to accept SSH from our server and web connection from
anywhere.

```
$ sudo iptables -A INPUT -p tcp --dport 22 -s <IP[/mask] -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

We used the following flags and parameters:

* `-p`: protocol
* `--dport`: port (works with the protocol)
* `-s`: source, mask is optional

We should also make sure that the machine can communicate with itself, so we
allow connections on the loopback device.

```
$ sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

We *insert* the rule in position 1 of the chain using `-I INPUT 1` to make sure
that this rule is evaluated first. We also specify the loopback device with `-i lo`.

There should be a rule that blocks connection that do not match explicit rules.
This can be achieved by specifying a default policy, although this could lead
to the server being unreachable if a rule allowing access is misconfigured or
accidentally deleted. The `-P` sets the default policy for a chain.

```
$ sudo iptables -P INPUT DROP
```

It is safer to have an open default policy, but set a blocking rule at the end
of the chain to catch connections that do not match any other rule.

```
$ sudo iptables -A INPUT -j DROP
```

To add a new rule at the end of the chain, one could delete the last rule,
add a new rule, add the blocking rule back.

```
$ sudo iptables -D INPUT -j DROP
$ sudo iptables -A INPUT ...
$ sudo iptables -A INPUT -j DROP
```

One can also *insert* the new rule in penultimate position. First, find the
relevant position by running:

```
$ sudo iptables -L --line-numbers
```

The configuration of `iptables` is ephemeral, meaning that all rules are
deleted when the server restarts. You can easily make your firewall
configuration persistent with the help of the `iptables-persistent` package. On
CentOS 6 and older you can also use the `iptables` init script to save rules in
the file `/etc/sysconfig/iptables`.

```
sudo service iptables save
```

More information can be found on [this
tutorial](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands).

### Miscellaneous

#### `tcpdump`

The `tcpdump` utility prints a description of packets on a network interface
matching provided arguments.

```
sudo tcpdump -n -i enp0s3 tcp port 22 and host 192.168.1.100
```

This command will show packets transiting on the interface `enp0s3` via the TCP
protocol on port 22, with the host 192.168.1.100 as its destination.

The `-A` argument will print packets in ASCII, this can be helpful.

#### Others

`nmap -v -sT` perform a verbose TCP scan
`iperf` to check performance of network connection
