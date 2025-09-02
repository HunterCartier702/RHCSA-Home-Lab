# RHCSA-Home-Lab

## Table of Contents

  - [Introduction](#intro)
  - [Basic Shell Skills](#basic)
  - [File Management Tools](#files)
  - [Managing Text Files](#awk)
  - [Local Consoles and SSH](#consoles)
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

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="files"></a>File Management Tools
Viewing Mounts

```shell
# a mount is a connection between a device and a directory
$ mount # lists all mounted devices and reads from /proc/mounts
$ df -Th # show available disk space on mounted devices
$ findmnt # show mounts and relationship between different mounts
```

Copy Files and Directories

```shell
$ ls -lrt # list recently modified files last 
$ cp /somedir/.* /tmp/ # copy all hidden files
$ cp -a /somedir/ /tmp/ # -a=archive. cp somedir and hidden and normal files to /tmp
$ cp -a /somedir/. . # copies all files regular and hidden to current dir
```

Using Links

```shell
$ touch file1; ln file1 file2 # create hard link
# cannot hard link to directories
# to create hard links you must be the owner of the item you want to link to
$ ln /etc/passwd . # this wont work as root owns this file
$ ln -s /etc/passwd . # creates a sym-link named passwd in current dir owned by current user
$ # unlink <file> or rm <file> to unlink. Just deletes the file
```

Archives and Compressed Files

```shell
# to archive a file you need read perms on it and execute perms on the directory the file resides in
$ mkdir testdir && touch testdir/file{1..100} # create junk in testdir
$ tar -cvf test-$(date +%F).tar testdir/ # archive testdir and files inside
$ tar -rvf existing.tar /etc/hosts # -r=append adds a file to an existing archive
$ tar -uvf existing.tar # update existing archive. write newer version of /home to archive
$ tar -tf existing.tar # to view contents of archive
$ file existing.tar # show what type of file
$ tar -xvf existing.tar # x=extract archive
$ tar -xvf existing.tar -C /tmp etc/hosts # unpack hosts file to /tmp
$ tar -xvf existing.tar etc/hosts # extract a only single file named hosts from archive
$ tar -cjvf test.tar.bz2 ./* # -j=bzip2. bzip2/bunzip2
$ tar -cJvf test.tar.xz ./* # -J=xz 
$ tar -czvf test.tar.gz ./* # z=gzip gzip/gunzip
```


[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="awk"></a>Managing Text Files
Cut

```shell
$ ps aux | less -N --use-color # pipe long output to less pager and -N for line numbering
$ sudo tail -f /var/log/messages # monitors the last 10 lines 
$ # cut: -d=field-delimiter -f="number of fields"
$ cut -d : -f 1 /etc/passwd | sort # show only users in byte order
$ cut -d : -f 3 /etc/passwd | sort -n # sort 3rd field numerically
$ du -h | sort -rn # sort largest files first
$ sort -k3 -t : /etc/passwd # sort third column w/ field-sep :
$ ps aux | sort -k 4 -nr | less # numerically reverse sort memory usage. 
$ ps aux | wc
		lines  words  chars
    200    2267   17520
```

Simple Regex

```shell
# safer to quote expressions
$ grep '^anna' /etc/passwd # search lines where anna is at the beginning
$ grep 'bash$' /etc/passwd # search lines that end w/ bash
$ grep 'r..t' /etc/passwd # '.' is a single wildcard
$ grep 'r[aut]t' /etc/passwd # matches a single char from the range of chars
$ ls -R Pictures/*.png # '*' match zero or more previous chars
$ grep 're\{2\}t' /etc/passwd # matches reed not red. If you know how many of the previous chars you want to match. 
$ grep 'ro\{1,3\}t' /etc/passwd # matchs min 1 max 3 of previous char
$ grep 'ro\?t' /etc/passwd # matches zero or one of the previous char
$ grep 'ro\+t' /etc/passwd # matches one or more of the previous char
$ (...) # group multiple chars so that regex can appy to the group
$ grep -v -e '^#' -e '^$' /etc/services | less # exclude blank lines and lines that start w/ #
```

awk and sed

```shell
$ awk -F : '{ print $4 }' /etc/passwd # prints only the 4th field
$ awk -F : '/user/ { print $4 }' /etc/passwd # print 4th field of any line w/ user
$ sed -n 5p /etc/passwd # print the 5th line in passwd file
$ sed -i 's/old/new/g' ~/newfile.txt # replace old text w/ new
$ sed -i -e '2d;20,25d' ~/newfile.txt # delete lines 2 and 20-25
$ sed -i -e '2d' ~/newfile.txt # del only line 2
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="consoles"></a>Local Consoles and SSH

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="summary"></a>Summary

Enter summary here: Still in progress...

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

