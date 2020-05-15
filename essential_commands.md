# Essential Commands

##  Log into local and remote graphical and text mode consoles

### User account management

#### Create user accounts

User information (user id, groups) can be displayed with the `id` command. For example, to show all group names that a user belongs to:

```
id -Gn <user>
```

To create a user, one can use `adduser` or the lower-level command `useradd`.

```
sudo adduser jaynee --home /home/jaynee --shell /bin/bash
```

The user will be prompted for a password an personal information. The user account can be modified by running `usermod`, for example changing the shell:

```
sudo usermod --shell /bin/sh
```

If you wanted to change your own shell, you could run `chsh -l` to see the list of available shells and then run `chsh -s /bin/sh`. You can see that the file `/etc/passwd` now shows your new shell, and on your next login you would enter the new shell.

The user will be added to the `/etc/passwd` file. This file usually does not contain password information, which is encrypted and stored in the restricted file `/etc/shadow`.

Changing the password of a user account is done with `passwd` (to change your own password you can omit the user login name):

```
sudo passwd jaynee pa$$w0rd
```

One can also change the password by piping it into `chpasswd`:

```
echo '<username>:<password>' | sudo chpasswd
```

Account age information can be viewed or modified with `chage`.  For example, to list the info:

```
sudo chage -l <username>
```

#### Delete a user account

To remove a user you can use:

```
sudo userdel -r <username>
```

With the argument `-r` the user's home directory and its files will be deleted, as well as the user's mail spool (the default one can be found in `/etc/login.defs`). Files in other filesystems have to be searched and deleted manually.

To delete any file this user owns, it's convenient to run:

```
sudo find /home -uid <user id> -delete
```

#### Defaults

The default values for commands such as `useradd`, `usermod`, `userdel`, `groupadd`, etc. are defined in `/etc/login.defs`.

Default values for `useradd` can be displayed using:

```
sudo useradd -D
```

They can also be found (and modified) in `/etc/default/useradd`.

### Log into a local account

You can log into an account with the `login` command (you'll be prompted for a password):

```
sudo login jaynee
```

One could also *switch user* with `su` but this does not open a login shell like `login` does, so a different environment will be loaded. Use `su -` (same as `su -l`) to use the target user environment.

```
su - jaynee
```

### Login scripts

We a user logs into an interactive session, scripts are run to prepare the environment. There is a difference between a **login shell** and a **non-login shell**.

#### Login shell

A login shell is launched when a user logs in with `ssh`, `su -`, `login`, etc. because login information is required (e.g. username and password).

In Ubuntu the following scripts will be executed:

* `/etc/profile` is a system-wide profile for Bourne-like shells
* `/etc/bash.bashrc` is system-wide, too
* all shell scripts in `/etc/profile.d/` (directory to put scripts to be executed instead of having to modify `/etc/profile`)
* `~/.bashrc` which is a user-specific profile

#### Non-login shell

Non-login shells are launched when one opens a new terminal, for example.

In Ubuntu the following scripts will be executed:

* `/etc/bash.bashrc`
* `~/.bashrc`


#### Home directory templates

Home environment profiles set for new users are defined in the directory `/etc/skel`.

### Log into a remote account

You can log into a remote machine with an SSH client using the command `ssh`. Use `ssh -i` to specify the private key which identifies you. For example:

```
ssh -i ~/.ssh/myprivatekey.pem <username>@<hostname>
```

You can also specify a per-host configuration in the `.ssh/config` file.

## Search for files

### `ls`

The command `ls` lists the current directory contents. Without any argument the objects are sorted alphabetically. Common options include:

* `-a` include files and directories starting with `.`
* `-h` with `-l` and/or `-s` to print file size in a human-readable format (shows number of bytes by default)j
* `-l` show contents as a long list
* `-r` reverse the sorting order
* `-S` sort by file size (largest first)
* `-t` sort by modification time (newest first)
* `-u` sort by access time (newest first), with `-l` show access time but sort by name, with `-lt` sort by and show access time

`ls` can be used with a file name pattern to list files which match some extension (or prefix). The following example will show only csv files:

```
ls *.csv
```

### `find`

The command `find` is a powerful tool to search files in any location. Its usage is:

```
find [-H] [-L] [-P] [-D debugopts] [-Olevel] [starting point] [expression]
```

Options `-H`, `-L` and `-P` control the treatment of symbolic links. `-D` is used to print diagnostic information to help debugging. `-O` is the optimization level (from 0 to 3). The starting point is the directory location where to start searching objects. Expressions provide arguments to filter the search according to a specific name, file size, etc.

For example, to find a file named *query.sql* in the Documents directory:

```
find ~/Documents -name query.sql
```

To find all csv files starting from the current directory:

```
find . -name "*.csv"
```

Find all png files (extension can be PNG or png) modified by the user `ubuntu` in the last 7 days, and ignoring the case of the name:

```
find /home/ubuntu -user ubuntu -mtime 7 -iname "*.png"
```

Other common expressions include:

* `-f` search files only
* `-d` search directories only
* `-maxdepth` only search until a specified directory depth
* `-not` negates the search parameter

The expression `-exec` can be used to execute some command on files found by `find`:

```
find ~/.ssh -f -name "*.pem" -exec chmod 400 '{}' \;
```

The previous command will make the private SSH keys read-only to the file owner and deny access for other users. The curly braces are a placeholder for the output of `find`. The are enclosed in single quotes to avoid handing `grep` a malformed file name. The `grep` command ends with a semi-colon, which has to be escaped with a backslash.

### `grep`

`grep` finds patterns in files and print matches.  If no file is given, recursive searches examine the directory while non-recursive searches examine the standard input. It's usage is:

```
grep [options] pattern [file ...]
```

The following command occurences of the word "python" in the script `user_data.sh`:


```
grep python user_data.sh
```

The option `-v` or `--invert-match` prints line which do not match a pattern. In the next command we pipe the output of `ps` to `grep` and remove the occurence of `grep` itself in the output:

```
ps ax | grep airflow | grep -v grep
```

To recursively search for a pattern through files of a directory, use the option `-r` or `--recursive`. With `-r`, symlinks are skipped. To follow them, use `-R` or `--dereference-recursive`:

```
grep -r python .
```


To show only the file name and not the matching pattern, use `-l` of `--file-with-matches`. Here we look for names of files which contain "python", amongst shell scripts:

```
grep -l python *.sh
```

Other options include:

* `-i` or `--ignore-case`
* `-w` or `--word-regexp` to search the pattern as a whole word
* `-n` or `--line-number` to show the line number of matchin patterns
* `-c` or `--count` to count matches
* `-q` or `--quiet` to not output matches, but just exit with status 0 if a match is found
* `-B n` prints n lines before a match
* `-A n` prints n lines after a match
* `-C n` prints n lines before and after a match

`grep` has three regular expression feature sets:

* basic (default)
* extended (called with options `-E`)
* Perl-compatible (called with `-P`)

## File system features

### Monitor storage devices

#### `df` and `du`: storage capacity and usage

The command `df` is used to check how much storage space is available in total and to see the current utilization of drives. The default output is displayed in 1K blocks, the `-h` flag makes it more human readable.

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.9G     0  2.9G   0% /dev
tmpfs           598M  1.6M  596M   1% /run
/dev/sda1        49G   23G   24G  50% /
```

One can exclude specific file systems with the `-x` option:

```
df -h -x tmpfs -x devtmpfs
```

`du` analyzes usage for the current directory and any subdirectory. Use the following options:

* `-h` human-readable format
* `-s` summarize results and show only grand total
* `-c` show individual results as well as total at the bottom

#### `lsblk`: fine information about block devices

A **block device** is a physical storage device where the file system is written, that reads and writes in blocks of a specific size, which applies to almost every type of non-volatile storage such as hard disk drives, solid state drives, etc.

Some systems require `sudo` for `lsblk` to display correctly:

```
sudo lsblk
```

The previous command gives the output:

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0  49.6G  0 disk 
└─sda1   8:1    0  49.6G  0 part /
sr0     11:0    1  1024M  0 rom
```

We have one disk (`/dev/sda`) with a single partition (`/dev/sda`) being used as the `/` partition.

Information relevant to the filesystem can displayed using the `--fs` option. This will show the filesystem type, the label and uuid.

### Working with filesystem mounts

Before one can use a new disk, it has to be partitioned, formatted with a filesystem, then the parititons or drive needs to be mounted. Partitioning and formatting are one-time processes, but mounting may be done frequently. Mounting the filesystem makes it available to the server at the selected mount point. A **mount point** is just a directory where the filesystem can be accessed.

We'll review the following commands:

* `mount` to attach a filesystem to the current file tree
* `umount` (not `unmount`) to detach a filesystem
* `findmnt` to gather info about the mounted filesystems

#### `mount`

`mount` is passed a formatted device or partition and a mount point (which should generally be an empty directory). The type of the filesystem can be specified with `-t`:

```
sudo mount -t ext4 /dev/sda1 /mnt
```

Mount options can be specified with `-o`. The following command mounts with default options and overrides read-write permissions and mounts as read-only:

```
sudo mount -t ext4 -o defaults,ro /dev/sda1 /mnt
```

#### `findmnt`

To display the mount options used for a specific mount, pass it to the `findmnt` command. For example, if we ran `lsblk` and we see than the filesystem `/dev/sda1` is mounted on `/`:

```
$ findmnt /
TARGET SOURCE    FSTYPE OPTIONS
/      /dev/sda1 ext4   rw,relatime,errors=remount-ro,data=ordered
```

#### `umount`

To unmount a filesystem, we simply pass the name of the mount point:

```
sudo umount /mnt
```

### Compare and manipulate file contents
