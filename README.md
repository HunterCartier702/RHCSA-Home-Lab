# RHCSA-Home-Lab

## Table of Contents

  - [Introduction](#intro)
  - [Basic Shell Skills](#basic)
  - [File Management Tools](#files)
  - [Summary](#summary)

## <a name="intro"></a>Introduction 
I decided to try my luck at the RHCSA exam. I decided to use a mix of VM’s on Virtual Box and physical hardware to study for the Red Hat certification. I will be documenting some of what I learn while doing this lab in hopes that it helps reinforce the knowledge. By copying down the commands I run and looking them over I know it will help me retain the material. For this project I bought a refurbished Lenovo for $120 dollars with these specs:

Lenovo ThinkCentre M910S Intel i7 3.20 GHz, 16GB DDR4 RAM, 1TB SSD, and I installed an extra 256GB SSD.

***Lenovo ThinkCentre***
<p align="center"><img alt="Tower" src="rhel_server/00Tower.jpeg" height="auto" width="800"></p>
<p align="center"><img alt="HTOP" src="rhel_server/01HTOP.jpeg" height="auto" width="800"></p>

***CPU***
<p align="center"><img alt="lscpu" src="rhel_server/03cpu.png" height="auto" width="800"></p>

***Memory and Storage***
<p align="center"><img alt="memory" src="rhel_server/02memory.png" height="auto" width="800"></p>

***Installed extra SSD***
<p align="center"><img alt="SSD" src="rhel_server/04SSDInstall.jpeg" height="auto" width="800"></p>


## <a name="basic"></a>Basic Shell Skills
Vim Basics

```shell
dd # delete current line into memory
yy # copies current line
p # pastes current line
v # visual mode and you can selece blocks of text. d to cut y to copy
u # undo last command
Ctrl+r # Redo last command. Only once
?text # search backwards 
^ # go to first position in current line
$ # go to end of line
:%s/old/new/g # replace all occurances of old w/ new. 'gc' to add confirmations
```

I/O Redirection

```shell
$ # STDIN 0<
$ # STDOUT 1>
$ # STDERR 2> 
$ ls wasd /etc 2>/dev/null > stdout.txt # discard stderr and save stdout to file
$ cat < /etc/passwd # send stdin to cat command
$ history | less # send stdout of history as stdin of less command
$ ls wasd /etc &> errout.txt # send both stderr and stdout to same file
```

History and Environment Variables

```shell
$ alias # view current aliases
$ which ls # find where shell will get ls command
$ type ls # find that ls is aliased to 'ls --color=auto'
$ history -d <number> # delete specfic number in history
$ history -c # clear current history in memory
$ history -w # clear current history file
$ env # list environment variables
```

Environment config files

```shell
$ /etc/profile # generic file processed by all users at login
$ /etc/bashrc # this file is processed when subshells are started
$ ~/.bash_profile # user specific login shell variables can be defined
$ ~/.bashrc # user specific subshell variables can be defined
# login shell is first shell opened for that user
```

/etc/motd and /etc/issue

```shell
$ /etc/motd # use to display a message after a user logs into a shell
$ /etc/issue # ues to display a message before a user logs in to a shell
```

Man Pages

```shell
# Man pages are categorized in different sections
# (1) Executable programs or shell commands
# (5) File formats and conventions
# (8) System administration commands
$ man 5 tar # access different sections
$ # append --help to most commands for brief usage
$ man ls # type /example for usage or look for See Also section
$ # serach the mandb with 'man -k' and 'apropos' using keywords
$ apropos partition
$ man -f ls
$ sudo mandb # update mandb
$ man ls 
	G # to go to bottom
	q # to exit
$ info '(coreutils) ls invocation' # read info page
$ ls /usr/share/doc # documentation files
```

## <a name="files"></a>File Management Tools
Viewing Mounts

```shell
# a mount is a connection between a device and a directory
$ mount # lists all mounted devices and reads from /proc/mounts
$ df -Th # show available disk space on mounted devices
$ findmnt # show mounts and relationship between different mounts
```


[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="summary"></a>Summary

Enter summary here: Still in progress...

