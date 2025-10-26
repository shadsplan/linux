# Calm lil tutorial

Following this: https://www.youtube.com/watch?v=v392lEyM29A

## Chapter 1 - Terminals and Shells
### Echo
``` bash
shad@linux:~/linux/tutorial$ name="shad"
shad@linux:~/linux/tutorial$ language="python"
shad@linux:~/linux/tutorial$ echo "My name is $name. My favorite language is $language"
My name is shad. My favorite language is python
```

### History
``` bash
shad@linux:~/linux/tutorial$ history | tail -n 4
  432  history
  433  history | top 4
  434  history | head -n 4
  435  history | tail -n 4
```
## Chapter 2 - Filesystems
- A file is a binary blob w/ metadata attached to them.
- A directory is a special type of file that contains references to other files and directories.

| btw, you can do `man curl` then do `/-L` and press n to find what you're looking for.
| also, `sudo apt install unzip`

``` bash
curl -L https://github.com/bootdotdev/worldbanc/archive/refs/heads/main.zip -o worldbanc.zip
unzip worldbanc.zip
rm worldbanc.zip
mv worldbanc-main worldbanc
sudo chown -R $(whoami) worldbanc
sudo chmod -R 755 worldbanc


shad@linux:~/linux/tutorial/2_filesystems$ ls -l
total 4
drwxrwxr-x 4 shad shad 4096 Jan 13  2025 worldbanc

# chmod is used to change file permissions (the stuff before the 4 above).
# chown is used to change file ownership (the stuff after the 4 above).
```

### more and less

``` bash
# casing matters
shad@linux:~/linux/tutorial/2_filesystems/worldbanc/private/transactions$ less -N 2023.csv
# when in interactive mode, do g153 to go to line 153
# it'll give you this:
153 876.65,12,2,Mia,Charlie,2023-07-30 03:44:00
```

### rm
I noticed that if the files in the directory are not write-protected (not read-only),  then `rm -r <directory>` works without the `-f` flag.

### copy
Use `-r` to copy directories and their contents. If you don't it will error out.

``` bash
shad@linux:~$ cp dir1 dir3
cp: -r not specified; omitting directory 'dir1'

shad@linux:~$ cp -r dir1/* dir2
shad@linux:~$ cd dir2
shad@linux:~/dir2$ ls
dir.txt
```

### home
In a Unix-like operating system, a user's home directory is the directory where their personal files are stored. It is also the directory that a user starts in when logging into the system.

`~` is the shortcut for the home directory, not the root directory `/`

### grep
Finding texts witin files.
``` bash
shad@linux:~/linux/tutorial/2_filesystems/worldbanc/private/logs$ grep -r "CRITICAL" .
./2024-01-13.log:2024-01-13 00:03:08 CRITICAL: Power to the main datacenter has been lost. Attempting to reconnect.
./2024-01-14.log:2024-01-14 00:02:58 CRITICAL: No one paid Dennis Nedry for his work. T-Rex released.
./2024-01-12.log:2024-01-12 00:04:07 CRITICAL: Boots is out of Salmon again.
./2024-01-11.log:2024-01-11 00:03:24 CRITICAL: Someone rm -rf'd the root directory.
./2024-01-11.log:2024-01-11 17:03:14 CRITICAL: Security scan is being maliciously manipulated.
./2024-01-16.log:2024-01-16 00:05:21 CRITICAL: Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0).
./2024-01-10.log:2024-01-10 01:10:27 CRITICAL: Boots is loose in the server room.
./2024-01-10.log:2024-01-10 05:58:18 CRITICAL: Server is on fire, please evacuate.
./2024-01-10.log:2024-01-10 20:13:04 CRITICAL: Someone rm -rf'd the root directory.
./2024-01-10.log:2024-01-10 20:19:37 CRITICAL: Database files are corrupt.
./2024-01-15.log:2024-01-15 00:03:57 CRITICAL: Worldbanc monitoring systems offline.
```

### find
Finding files themselves.
``` bash
shad@linux:~/linux/tutorial/worldbanc$ find public/products -name "*joint*"
public/products/loans/jointloanshark.txt
public/products/credit_cards/jointrewardsplux.txt
public/products/accounts/jointsavings.txt
public/products/accounts/jointchecking.txt
```

## Chapter 3 - Permissions
Unix-like systems (like the one you're using) support multiple users. Each user has their own home directory, their own files, and their own permissions.

The sudo keyword lets you run a command as a "superuser". It's short for "superuser do". To use it, you'll need a password with superuser privileges, which you should already have if you're the only user of your machine.

Permissions:
1. Who has the permission?
2. What permission do they have?

Who/What are represented by a 10-character string:
- 1st character - file type (`-` for regular files, `d` for directories)
- Next 3 characters - owner's permissions for read (r), write (w), execute (x). Commonly the user who created the file.
- Next 3 characters - group's permissions for read (r), write (w), execute (x). Commonly a group of users.
- Last 3 characters - others' permissions for read (r), write (w), execute (x). Everyone else.

drwxr-xr--: 
- d: directory
- rwx: owner can read, write, execute
- r-x: group can read, execute
- r--: others can read only

``` bash
shad@linux:~/linux/tutorial/worldbanc$ ls -l private/
total 24
drwxr-xr-x 2 shad shad 4096 Oct 24 23:10 bin
drwxr-xr-x 4 shad shad 4096 Oct 24 23:10 cmd
drwxr-xr-x 2 shad shad 4096 Oct 24 23:10 contacts
drwxr-xr-x 2 shad shad 4096 Oct 24 23:10 customers
drwxr-xr-x 2 shad shad 4096 Oct 24 23:10 logs
drwxr-xr-x 3 shad shad 4096 Oct 24 23:10 transactions
shad@linux:~/linux/tutorial/worldbanc$ chmod -R 700 private/
shad@linux:~/linux/tutorial/worldbanc$ ls -l private/
total 24
drwx------ 2 shad shad 4096 Oct 24 23:10 bin
drwx------ 4 shad shad 4096 Oct 24 23:10 cmd
drwx------ 2 shad shad 4096 Oct 24 23:10 contacts
drwx------ 2 shad shad 4096 Oct 24 23:10 customers
drwx------ 2 shad shad 4096 Oct 24 23:10 logs
drwx------ 3 shad shad 4096 Oct 24 23:10 transactions
shad@linux:~/linux/tutorial/worldbanc$ ls -l
total 12
-rwxr-xr-x 1 shad shad  145 Oct 24 23:10 README.md
drwx------ 8 shad shad 4096 Oct 24 23:10 private
drwxr-xr-x 3 shad shad 4096 Oct 24 23:10 public
```

### Executables

We can also simply remove/add executable permission from a file:

``` bash
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ls -l
total 20
-rwx------ 1 shad shad 209 Oct 24 23:10 genids.sh
-rwx------ 1 shad shad 753 Oct 24 23:10 malicious.sh
-rwx------ 1 shad shad 555 Oct 24 23:10 process_transactions.sh
-rwx------ 1 shad shad 616 Oct 24 23:10 warn.sh
-rwx------ 1 shad shad 339 Oct 24 23:10 worldbanc.sh

shad@linux:~/linux/tutorial/worldbanc/private/bin$ chmod -x genids.sh 
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ls -l
total 20
-rw------- 1 shad shad 209 Oct 24 23:10 genids.sh
-rwx------ 1 shad shad 753 Oct 24 23:10 malicious.sh
-rwx------ 1 shad shad 555 Oct 24 23:10 process_transactions.sh
-rwx------ 1 shad shad 616 Oct 24 23:10 warn.sh
-rwx------ 1 shad shad 339 Oct 24 23:10 worldbanc.sh
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ./genids.sh
bash: ./genids.sh: Permission denied

shad@linux:~/linux/tutorial/worldbanc/private/bin$ chmod +x genids.sh 
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ls -l
total 20
-rwx--x--x 1 shad shad 209 Oct 24 23:10 genids.sh
-rwx------ 1 shad shad 753 Oct 24 23:10 malicious.sh
-rwx------ 1 shad shad 555 Oct 24 23:10 process_transactions.sh
-rwx------ 1 shad shad 616 Oct 24 23:10 warn.sh
-rwx------ 1 shad shad 339 Oct 24 23:10 worldbanc.sh
```

### chown
`chmod` allows you to change the permissions of any file or directory that you own. But what if you don't own the file or directory? That's where `sudo` is required. 

The chown command, which stands for "change owner", allows you to change the owner of a file or directory, and it requires root privileges.

## Chapter 4 - Programs

A program is just a set of instructions that a computer can execute, and an "executable" is just a file that contains a program. 

- Compiled Program: is a program that has been converted from human-readable source code into machine code (binary). Machine code is a set of instructions that a computer can execute directly: your computer's CPU is hardware that's been designed to execute machine code.

- An interpreted program is a program that is executed by another program. The program that executes the interpreted program is called an interpreter. The interpreter reads the source code of the interpreted program and executes it. Your computer needs to have the interpreter installed to run the program.

The `which` command tells you the location of an installed command line program.

Bash is an interpreted language. When you run `./script.sh`, the bash interpreter (`/usr/bin/bash`) reads and executes each line one at a time.

It leverages the `#!/bin/bash` to know which interpreter to use. No binary compilation is needed.

### Shell Configuration

`ls -a ~` to see hidden files in your home directory.

### Environment Variables

- `env` to see all environment variables.

```
shad@linux:~/linux/tutorial/worldbanc/private/bin$ export WARN_MESSAGE="The auditor is here"
shad@linux:~/linux/tutorial/worldbanc/private/bin$ export WARN_FROM_NAME="Your worst nightmare"
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ls -l worldbanc.sh 
-rwx------ 1 shad shad 339 Oct 24 23:10 worldbanc.sh
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ./warn.sh 
============================================
=========== WORLDBANC WARNING ==============
============================================
The auditor is here
============================================
From: Your worst nightmare
============================================
```

Note: The `export` command only sets the environment variable for the current terminal session. If you open a new terminal, the variable will not be set. To make it permanent, you need to add the export command to your shell configuration file (ex. `~/.bashrc`).

To have it immediately available in the current session, you can `source ~/.bashrc` after adding it.

### PATH

The PATH is an environment variable that stores a list of directories. Any file in any directory in that list, can be run straight from the command line without having to type out the full path to the file.

``` bash
shad@linux:~/linux/tutorial/worldbanc/private/bin$ which cat
/usr/bin/cat
shad@linux:~/linux/tutorial/worldbanc/private/bin$ echo $PATH
/home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/bin/remote-cli:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/shad/.vscode-server/data/User/globalStorage/github.copilot-chat/debugCommand
```

When I type `cat`, without typing the full absolute file path `/usr/bin/cat`, it's because `/usr/bin` is being scanned for the `cat` executable.

When you install programs that you want to be able to use system-wide, just using their name, the directory that program is in needs to be in your PATH.

`export PATH="$PATH:/path/to/new"` to add a directory to your PATH without overriding all of the existing directories.

## Chapter 5 - Input/Output

### man
`man` helpful scrolling tips:
- Press `Space` to go down one page, `b` to go back one page.
- Press `u` to go up half a page, `d` to go down half a page.
- Press `g` to go to the beginning of the document, `G` to go to the end.
- Press `/keyword` to search for "keyword" forward, `?keyword` to search backward.
- Press `n` to go to the next search result, `N` to go to the previous search result.
- Press `q` to quit.

### flags

Common flag conventions:
- Single-character flags are prefixed with a single dash (.e.g -a)
- Multi-character flags are prefixed with two dashes (e.g. --help)

```
shad@linux:~$ wc -c ~/linux/tutorial/worldbanc/public/pr_ideas.txt 
1098 /home/shad/linux/tutorial/worldbanc/public/pr_ideas.txt
```

### help
By convention, most production-ready CLI tools have a "help" option that prints information about how to use the tool. Either `-h` or `--help` will usually work.

This is sometimes easier to parse then the full `man` page.

- `0` is the exit code for success. Any other exit code is an error.
- `echo $?` to see the exit code of the last command.

``` bash
shad@linux:~/linux/tutorial$ ls -l
total 16
-rw-rw-r-- 1 shad shad 11802 Oct 25 19:39 LEARN.md
drwxr-xr-x 4 shad shad  4096 Oct 24 23:10 worldbanc
shad@linux:~/linux/tutorial$ echo $?
0
shad@linux:~/linux/tutorial$ ls -yooo
ls: invalid option -- 'y'
Try 'ls --help' for more information.
shad@linux:~/linux/tutorial$ echo $?
2
```

### Redirecting Streams
- You can redirect stdout and stderr to different places using the `>` and `2>` operators. `>` redirects stdout, and `2>` redirects stderr.

### Standard Input
- `READ` command in a bash script will read from standard input and store it in a variable.

### Piping

One of the most beautiful things about the shell is that you can pipe the output of one program into the input of another program.

```bash

# Incorrect, it matched the pattern transactions/backups/ in transactions/ which is not what we want.
shad@linux:~/linux/tutorial/worldbanc/private$ grep -r "Bob" transactions/ --exclude-dir=transactions/backups/ | wc -l
6975


shad@linux:~/linux/tutorial/worldbanc/private$ grep -r "Bob" transactions/ --exclude-dir=backups | wc -l
3973
```

### kill

``` bash
shad@linux:~/linux/tutorial/worldbanc/private/bin$ ./malicious.sh force
SIGINT ignored due to 'force' argument
Doing scary stuff...
Doing scary stuff...
^C
HAHA! You can't stop me!
Doing scary stuff...
Doing scary stuff...
^C
HAHA! You can't stop me!
Doing scary stuff...
Doing scary stuff...
...
# in a different terminal
shad@linux:~$ ps aux | grep "malicious"
shad        9209  0.0  0.0   7740  3592 pts/0    S+   20:44   0:00 bash ./malicious.sh force
shad        9531  0.0  0.0   7080  2252 pts/1    S+   20:44   0:00 grep --color=auto malicious

# Not the second one, that's just the grep command itself.
# The first one is the actual malicious.sh script.
shad@linux:~$ kill 9209
...
Doing scary stuff...
Doing scary stuff...
Terminated
shad@linux:~/linux/tutorial/worldbanc/private/bin$ 
```

### Unix Philosophy

1. Write programs that do one thing and do it well.
2. Write programs to work together.
3. Write programs to handle text streams, because that is a universal interface.

### top

By default, `top` sorts the processes by CPU usage, with the most CPU-intensive processes at the top. Another useful resource to sort by is memory (RAM) usage. To sort by memory usage, press `M` (uppercase) while `top` is running. 

# Chapter 6 - Packages

- APT, or "Advanced Package Tool", is the primary package manager for Ubuntu.
- When running a command like `curl -sS https://webi.sh/lsd | sh; \`, you're downloading a script from the internet and executing it with `sh`.
  - This can be potentially dangerous if you don't trust the source of the script, as it could contain malicious code.
  - You can retrieve and review the script first, or in the case of programs, you typically want them signed by the publisher of the script to make sure it hasn't been tampered with.
  - btw https prevents mitm attacks.