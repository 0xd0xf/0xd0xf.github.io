---
layout: post
title: "Linux enumeration and privesc tricks: Commands & Files"
categories: [ENUMERATION,LINUX,PRIVESC]
---

I will use this post as a list of different ways to find information with a **bash shell** using command-line tools like grep or find. Also there is info to upgrade a shell.

## Files

```bash
# Lines from a file
cat file | wc -l

# Words from a file
cat file | wc -w

# Remove lines containing word
grep -Ev "word"

# Extract mail from files
grep -E -o "\b[a-zA-Z0-9.#?$*_-]+@[a-zA-Z0-9.#?$*_-]+.[a-zA-Z0-9.-]+\b" file

# Redirect stderr to null
2>/dev/null

# Find only files
find /folder -type f 2>/dev/null

# Find setuid files
find / -perm -4000 -type f 2>/dev/null

# Find root setuid files
find / -perm -4000 -uid 0 -type f 2>/dev/null

# Find writable directories
find / -perm 777 2>/dev/null
find / -writable -type d 2>/dev/null

# Find files from a user
find / -user user 2>/dev/null

# Cut files into columns by delimitters
cat file | cut -d"<Delimiter>" -f<columns>

# Join files (To correlate data and time for example)
paste file1 file2

# Find strings into files
grep -i string file

# Filter php files (Or other extensions)
find . -name "*.php"
```


## Info to check

```bash
# User info
id
who
sudo -l

# System version/release
cat /etc/lsb-release #Debian
cat /etc/redhat-release #Redhat
cat /etc/*-release #General based

# Kernel
uname -a
cat /proc/version

# Enviromental (Sometimes even command history)
cat /etc/bashrc
cat ~/.bashrc
cat ~/.bash_history
cat ~/.bash_profile
env

# Applications
ls -la /usr/bin
ls -la /sbin/

# Scheduled jobs
crontab -l
crontab -e
cat /etc/cron*

# Sensitive files
cat /etc/passwd
cat /etc/sudoers
cat /etc/shadow

# SSH Keys
ls -la ~/.ssh/

# List directories recursively
ls -laR directory
```


## Advanced

```bash
# Service reconfiguration
ls -aRl /etc/ | awk '$1 ~ /^.*w.*/' 2>/dev/null     # Anyone
ls -aRl /etc/ | awk '$1 ~ /^..w/' 2>/dev/null       # Owner
ls -aRl /etc/ | awk '$1 ~ /^.....w/' 2>/dev/null    # Group
ls -aRl /etc/ | awk '$1 ~ /w.$/' 2>/dev/null        # Other
find /etc/ -readable -type f 2>/dev/null            # Anyone

# Check filesystems
cat /etc/fstab
df -h

# Advanced permissions
find / -perm -1000 -type d 2>/dev/null   # Sticky bit - Only the owner of the directory or the owner of a file can delete or rename here.
find / -perm -g=s -type f 2>/dev/null    # SGID (chmod 2000) - run as the group, not the user who started it.
find / -perm -u=s -type f 2>/dev/null    # SUID (chmod 4000) - run as the owner, not the user who started it.

# Tools/Langs installed
find / -name python*
find / -name perl*
find / -name gcc*
find / -name wget
find / -name nc
find / -name ftp
```


## Shell upgrading
```bash
## Better shell
python -c 'import pty;pty.spawn("/bin/bash")'
/bin/sh -i
perl --e 'exec "/bin/sh";'

## On your machine, view parameters (On the term you are going to connect to server)
echo $TERM # Take note
stty -a # Take note (rows and columns)
stty raw -echo

# Then connect to the shell and issue this:
reset
export SHELL=bash
export TERM=xterm-256color #Or your term value from before
stty rows <rows> columns <cols> # Also from your before values
```

### Notes about stty command
> The dash means "disable" a setting. So this enables echoing:<br>
stty echo<br>
This disables it:<br>
stty -echo<br>
When you disable it, your typing is not echoed back to you, which is why it seems as if the terminal is hanging.<br>
Try stty -echo then type ls and press return - you will still see the output of ls.<br>
The raw setting means that the input and output is not processed, just sent straight through.<br>
Processing can be things like ignoring certain characters, translating characters into other characters, allowing interrupt signals etc.<br>
So with stty raw you cant hit Ctrl-C to end a process, for example.


## Resources / Links
### Scripts / Tools
[Script unix-privesc PentestMonkey](http://pentestmonkey.net/tools/audit/unix-privesc-check)

[LinEnum.sh](https://github.com/rebootuser/LinEnum)

[Enum 4 Linux](https://labs.portcullis.co.uk/tools/enum4linux/)

[pspy - Monitor Linux processes](https://github.com/DominicBreuker/pspy)


### Links / Info
[Basic guide](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)