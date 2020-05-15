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
