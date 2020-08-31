# Networking

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
