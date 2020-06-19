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

## Compare and manipulate file contents

### Compare files

#### `cmp`: compare files byte by byte

The `cmp` command compares files byte by byte. It will return 0 if the files
are the same and 1 if files are different. This is often useful in an `if else`
statement:

```
cmp -s <file 1> <file 2>
if [[ $? -ne 0 ]]; then
    echo "files are different"
else
    echo "files are the same"
fi
```

The `-s` option makes the command silent (output is only exit code). Other
options include:
- `-b`: print differing bytes
- `-i`: ignore a number of initial bytes
- `-l`: verbose

#### `diff`: compare two files line by line

The output of `diff` indicates how the lines in each file are different, and
the steps involved to change one file into the other. The output has the format
<range>[acd]<range> where the range is line numbers, while the middle character
indicates the action: *add*, *change* or *delete*.

Here is `file1.txt`:

```
my name is jonathan
i am french
i live in jamaica
the is the penultimate line
end of file
```

Here is `file2.txt`:

```
my name is jaynee
i am american
i live in jamaica
end of file
```

The command `diff file1.txt file2.txt` will return the following output:

```
1,2c1,1
< my name is jonathan
< i am french
---
> my name is jaynee
> i am american
4d3
< penultimate line
```

The following options can be used with `diff`:
- `-q`: brief output
- `-y`: show output side by side (`|` indicates a change, `<` or `>` a deletion)
- `-s`: report identical files (identical files lead to no output by default)

To compare the contents of directories, use `diff -r <dir 1> <dir 2>`

#### `uniq`: filter repeated lines

The `uniq` command eliminates consecutively repeated lines from a file:

```
$ uniq <file>
```

#### `vimdiff`: edit files in parallel

To open two files with `vim` for editing, run:

```
$ vimdiff <file 1> <file 2>
```

You can navigate to a different window like you would do normally with vim:
`Shift + w + [hjkl]`.

By default the split is vertical. To split the window horizontally, use the
`-o` option.

You can quit all splits with `:qa`.

### Modify file contents

#### `sed`: stream editor

`sed` is a non-interactive *stream editor*. It receives text input (`stdin` or
a file), performs operations on it one line at a time, then outputs the results
(`stdout` or a file). `sed` is often one of several components in a pipe. `sed`
determines which lines of the input it will operate on from the *address range*
passed to it. You can specify the address range either by line number or by a
pattern to match. Here are the basic `sed` operators:

- [address-range]/p: print
- [address-range]/d: delete
- s/pattern1/pattern2/: substitute pattern 2 for first instance of pattern 1 in
  a line
- [address-range]/s/pattern1/pattern2/: substitude pattern 2 for first instance
  of pattern 1 in a line, over *address-range*
- [address-range]/y/pattern1/pattern2/: replace any character in pattern 1 with
  the corresponding character in pattern 2, over address range (equivalent to
  `tr`)
- [address] i pattern filename: insert pattern at address in file, usually used
  with `-i` (in-place) option
- g: operate on every pattern match withing each matched line of input

For example, to print lines starting with the word "error" from the file
log.txt:

```
$ sed -n '/^error/p' log.txt
```

The option `-n` allows to print only the matching lines. The single-quotes are useful if this command is run in a script, to reserve the regular expression interpretation for `sed`. If the pattern was a variable which needs to be expanded, we would use double-quotes.

Examples:

```
# print 4th line
$ sed -n 4p <file>

# delete 8th line
$ sed 8d <file>

# delete blank lines
$ sed /^$/d <file>

# print lines containing dog
$ sed -n /dog/p <file>

# replace every instance of Windows by Linux
$ sed s/Windows/Linux/g <file>

# delete space at end of every line
$ sed s/ *$// <file>

# delete lines containing Mac
$ sed /Mac/d <file>

# delete instances of Mac, leaving remainder of each line intact
$ sed s/Mac//g <file>
```

## Input and output redirection

### Standard output

Many programs send their results to the *standard output* (stream number 1), which by default direct its contents to the display. Use `>` to redirect the standard output to
a file:

```
ls > file.txt
```

The `>` character will overwrite the file if it exists. Use `>>` to append to a
file, it will be create if it does not exist.

```
ls >> file.txt
```

### Standard input

Many commands accept input from the *standard input* (stream number 0). By default, standard input gets its contents from the keyboard but it can be redirected. To redirect
input from a file use the `<` character. In the following example we use the
`wc` command to count the number of lines in a file.

```
wc -l < file.txt
```

We can also redirect the output of `wc` to another file:

```
wc -l < file.txt > count.txt
```

Make sure that the redirection operators appear after the options and arguments
of the command.

### Pipelines

You can connect multiple commands together with *pipelines*, which feed the
output of one command into the input of another one. For example, you can
paginate results of a command using `less`:

```
ls -lh | less
```

Or perform a search on the output of a command:

```
ps -ef | grep python
```

Or display only part of a long list of results, for example the 10 newest files
in the current directory:

```
ls -lt | head
```

Here we display the list of directories and how much space they use:

```
du -h | sort -nr
```

The next command displays the total number of files in the current directory
and all of its subdirectories:

```
find . -type f | wc -l
```

### Standard error

The errors generated by a program are written to the *standard error* (stream
number 2), which by default is sent to the terminal display. The following
command redirects the standard error stream of a command to a log file:

```
bash script.sh 2> log.txt
```

### tee

The `tee` command redirects the output of a command to a file but also displays
the output on the terminal:

```
cat /etc/passwd | grep jonathan | tee results.txt
```

### Discard the output

Sometimes we don't want to collect the output of a command, so we can redirect
it to `/dev/null`, a special file that automatically discards its input.

To direct both the output of a command and its errors, we can redirect the
standard error to standard output:

```
<command > /dev/null 2>&1
```

### heredoc

We can pass multiple lines of text to a command using the following syntax:

```
[<command>] <<[-] 'DELIMITER'
type your
text here
DELIMITER
```

A few things to note:
- the initial command is optional
- the delimiter is usually EOF or END, by convention
- if the delimiter is unquoted, the shell will substitute variables, commands
  and special characters before passing the input lines to the command
- append a `-` (minus sign) to the redirection operator (`<<-`) will cause all
  leading tab characters to be ignored, so we can use indentation (only tab is
  allowed)

For example:

```
if true; then
    cat <<- EOF
    This line has a leadin tab.
    EOF
fi
```

The here-doc pattern is often used to redirect lines of input to a file:

```
cat << EOF > info.txt
The current working directory is $PWD
You are logged in as $(whoami)
EOF
```

We could redirect the previous example into a script by single-quoting the
delimiter to avoid variable and command interpretation:

```
cat << 'EOF' > script.sh
echo "The current working directory is $PWD"
echo "You are logged in as $(whoami)"
EOF
```

Here-doc is a convenient way to execute multiple commands on a remote system
over SSH:

```
ssh -T user@host << EOF
echo "the remote working directory is \$PWD"
EOF
```

## Analyze text with regular expressions

The `grep` commands is useful to search for patterns in a text (regular
expressions). `grep` stands for *global regular expression print*.

### Basic usage

You can print every line of a file containing a literal pattern:

```
grep "GNU" /usr/share/common/GPL-3
```

Common options include:

* `-i` ignore case
* `-v` match lines which **do not** contain the pattern
* `-n` print the lines the matches occur on
* `-r` recursively search files under each directory
* `-c` count the number of matches
* `-w` match whole words

It is also possible to use `grep` without a file. In this case, non-recursive
calls process the standard input and recursive calls process files in each
directory.

### Regular expressions

The anchor characters `^` (beginning of the line) and `$` (end of the line) specify where in the line a match must occur
to be valid.

You can match any character using `.`.

By placing a group of character within bracket `[` and `]`, you specify that
the character at this position can be any character of the bracket group. For
example, the following regex will match *too* and *two* which form whole words:

```
grep -w "t[wo]o" file.txt
```

By beginning a bracket group with `^`, you specify a pattern which matches
**anything except** the characters between brackets. The following command will
match any word ending with *ode* except *code* (but will match *Code*):

```
grep -w "[^c]ode" file.txt
```

You can also specify ranges within brackets, such as `A-Z`, `a-z` or `0-9`.
Note that POSIX character classes can also be used, for example `[:upper:]` is
equivalent to `A-Z`. Other classes include:

* `[:lower:]` for `a-z`
* `[:alpha:]` for `A-Za-z`
* `[:digit:]` for `0-9`
* `[:alnum:]` for `A-Za-z0-9`
* `[:space:]` for `\t`, `\n`, `\r`, `\f` (formfeed) or `\v` (vertical tab)

To escape meta-characters, for example if you want to search for a period or a
bracket, you need to escape these characters using a backslash `\`.

### Extended regular expressions

`grep` can be used with the `-E` flag to allow the use of extended regular
expressions, which allow the use of additional meta-characters for more complex
matches. One can also use `egrep`.

#### Grouping

You can group together expressions and manipulate them as a unit with the use
of parentheses `()`, for example to reference the same group later in the expression. The following will match words which start and endt with the letters *te*. We use the *backreference* `\1` to reference the first group again.

```
grep -Ew "(te)[^ ]\1" file.txt
```

#### Alternations

Alternations use the *pipe* `|` character and allow you to specify several possibilities which should be considered a match:

```
grep -E "(GPL|General Public License)" GPL-3
```

#### Quantifiers

Quantifiers allow you to specify the number of occurences of characters:

* `?` 0 or 1 times
* `*` 0 or more
* `+` 1 or more

To specify the number of times a match should be repeated, use the brace
characters `{}`. You can specify an exact number, a range, an upper bound or a
lower bound:

* {3} specifies a match repeated exactly three times
* {3,6} repeated between three and six times
* {5,} repeated at least five times
* {,4} repeated at most four times

## Archive, backup, compress, unpack and uncompress files

## Create, delete, copy and move files and directories

## Create and manage soft and hard links

## List, set and change standard file permissions
