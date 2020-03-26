# Essential Commands

##  Log into local and remote graphical and text mode consoles

### Create and modify a user

To login as a specific user, this user has to be created. To create a user, one can use `adduser` or the lower-level command `useradd`.

```
sudo adduser jaynee --home /home/jaynee --shell /bin/bash
```

The user will be prompted for a password an personal information. The user account can be modified by running `usermod`, for example changing the shell:

```
sudo usermod --shell /bin/sh
```

Changing the password of a user account is done with `passwd`:

```
sudo passwd jaynee pa$$w0rd
```

To change your own password you can omit the user login name.


### Log into a local account

You can log into an account with the `login` command (you'll be prompted for a password):

```
sudo login jaynee
```

One could also *switch user* with `su` but this does not open a login shell like `login` does, so a different environment will be loaded. Use `su -` to use the target user environment.

```
su - jaynee
```

### Log into a remote account

You can log into a remote machine with an SSH client using the command `ssh`. Use `ssh -i` to specify the private key which identifies you. For example:

```
ssh -i ~/.ssh/myprivatekey.pem <username>@<hostname>
```

You can also specify a per-host configuration in the `.ssh/config` file.

## Search for files

### Search files based on name or other metadata

#### `ls`

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

#### `find`

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

#### `grep`

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
