# RHCSA-Home-Lab

## Table of Contents

  - [Introduction](#intro)
  - [Basic Shell Skills](#basic)
  - [File Management Tools](#files)
  - [Managing Text Files](#awk)
  - [Local Consoles and SSH](#consoles)
  - [Users and Groups](#sudo)
  - [Permission Management](#perms)
  - [Configuring Networking](#networking)
  - [Managing Software](#dnf)
  - [Managing Processes](#procs)
  - [Working with Systemd](#systemd)
  - [Scheduling Tasks](#tasks)
  - [Summary](#summary)

## <a name="intro"></a>Introduction 
I decided to study for the RHCSA exam. This is a compilation of notes that I took during my studies. For my labs I decided to use a mix of VM’s on Virtual Box and physical hardware to study for the Red Hat certification. I will be documenting some of what I learn while doing these labs in hopes that it helps reinforce the knowledge. By copying down the commands I run and looking them over I know it will help me retain the material. For this project I bought a refurbished Lenovo for $120 dollars with these specs:

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
Non-graphical Environments

```shell
# Virtual terminals allow you to open 6 different terminal windows on the same console.
# Press Alt+F1-6 to switch between consoles or use chvt.
# Ctrl+Alt+fn+F7 is the graphical console in graphical environments
$ sudo chvt 1 # Press 1-6 to switch. 7 for graphical console if there is one
```

Reboot and Poweroff

```shell
$ sudo systemctl reboot 
$ sudo systemctl halt
$ sudo systemctl poweroff
or 
$ sudo reboot # symlinked to /bin/systemctl
$ ls -l $(which reboot)
$ echo b > /proc/sysrq-trigger # from root shell to reset machine. Use as last resort
```

SSH and Related Utilities

```shell
$ sudo systecmctl status sshd # check if service is running
$ sed -i -e '25d' ~/.ssh/known_hosts # remove fingerprint if misatch from changes
$ ssh -Y user@server # connect to server and start graphical applications assuming X server is running and allowed
$ ssh -p 2222 user@server # change port
```

SCP Secure Copy

```shell
$ scp /etc/hosts rhelsvr01:/tmp # connect using current user acc to transfer file
$ scp cartier@rhelsvr01:/etc/hosts . # connect as user cartier and transfer hosts file to current directory
$ scp -r rhelsvr01:/etc/ /tmp # recursively copy subdirectory to local tmp dir
$ # -P to specify non-default port
$ sftp cartier@rhelsvr01 # enter password and access files with interactive prompt
	ls
	pwd # show current remote dir
	lpwd # show local current dir
	lcd /tmp # change local dir
	put secret_file.txt # upload a file
	exit
```

Key-based Auth

```shell
$ ssh-keygen # generate keys and run through prompts
$ ssh-copy-id rhelsvr01 # copy .pub key to remote server
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="sudo"></a>Users and Groups
sudo and visudo

<p align="center"><img alt="sudo" src="rhel_server/05visudo.png" height="auto" width="800"></p>

```shell
# 1. First field is user and % prefix is for Groups.
# 2. ALL= means all hosts. (useful in distributed environments with shared sudoers files).
# 3. User:Groups. Root user can run cmds as all users. tux can run cmds as root. If blank the root is implied
# 4. Commands to which the rule applies. Use absolute path and seperate by a comma. An ! mark before command name negates or forbids the user from running it
# Dan can run all commands without being prompted for sudo passwd
# NOEXEC prevents spawning other commands with same privs

$ sudo visudo # to edit /etc/sudoers file
# or add a file to /etc/sudoers.d/ for a user 
# files in sudoers.d/ must be owned by root with 0440 perms
#/etc/sudoers.d/jdoe
$ which systemctl
$ jdoe ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart apache2
$ linda ALL=/usr/bin/useradd, /usr/bin/passwd,! /usr/bin/passwd root
$ pkexec visudo # in case you lock root account
```

```shell
$ sudo -i
$ useradd betty; useradd amy
$ echo password | passwd --sdtin betty; echo password | passwd --stdin amy
$ exit
$ sudo sh -c 'echo "amy ALL=/usr/bin/useradd, /usr/bin/passwd, ! /usr/bin/passwd root" > /etc/sudoers.d/amy'
$ su - amy
$ sudo passwd betty
	success
$ sudo passwd root
	failure
```

User Accounts

```shell
# vipw=edit passwd, vipw -s=edit shadow, and vigr=edit group. Not recommended
$ sudo useradd -m -d /home/james -c “IT Guy” -s /bin/bash -G sudo,adm,mail james #end command with username. -m=creates home dir. -d=specify dir to put home dir. -s=default shell. -g=add user to another primary group, not common. -G=adds user to secondary group
# the /etc/skel dir contains files that are copied to the user home dir when created
# default shell for users is /bin/bash and system users is /sbin/nologin
# use usermod or passwd -s to change this
$ sudo useradd james -s /sbin/nologin # change shell from /bin/bash
$ usermod # use for modifying user properties execpt passwords. use passwd command
```

Configuration Files for Users

```shell
# when using useradd some default values are assumed in two config files:
$ /etc/default/useradd # contains the SKEL and SHELL vars
$ /etc/login.defs 
# Important default vars in login.defs:
# MOTD_FILE, ENV_PATH, PASS_MAX_DAYS, PASS_MIN_DAYS, PASS_WARN_DAYS, UID_MIN, CREATE_HOME
```

Managing Password Properties

```shell
# set password for james to min usage period of 30 days, expire after 90 days, and warning berfore 3 days:
$ sudo passwd -n 30 -w 30 -x 90 james
$ sudo chage -E 2025-12-31 james # set expire date
$ chage -l james # see current passwd settings
$ chage james # enter interactive mode
# interactive mode options:
Minimum Password Age [0]: 30
The minimum number of days a user must wait after changing their password before they can change it again.

Maximum Password Age [99999]: 180
The maximum number of days a password is valid before the system forces the user to change it.

Last Password Change (YYYY-MM-DD) [2025-07-08]:
The last time the password was changed.

Password Expiration Warning [7]: 14
How many days before the password expires the user gets a warning.

Password Inactive [-1]: 3
Number of days after a password expires before the account is disabled (i.e., login not allowed).

Account Expiration Date (YYYY-MM-DD) [-1]:
The date the account itself expires, not just the password.
Default: -1 means the account never expires.
```

Creating User Environment

```shell
 # when a user logs in an environment is created. These files play a role:
 /etc/profile # Used for default setting for all users when starting a login shell
 /etc/bashrc # Used to define defaults for all users when starting a subshell
 ~/.profile # specific settings for one user applied when starting a login shell
 ~/.bashrc # specific settings for one user applied when starting a subshell
 # When logging in these files are read in this order. 
 # Variables and other settings are defined and applied from these files
 # If a variable or setting occurs in more than one setting, the last one wins
```

Managing Group Accounts

```shell
/etc/group # only shows names of users who are a member of this as a secondary group
$ sudo groupadd ssh_users # create a group
$ groupmod # use to change group name or gid, but not change group members, use usermod
$ lid -g wheel # check members of a group
$ groupmems -g wheel -l # show secondary and primary group users
$ groupmod -aG ssh_users linda
$ id linda
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="perms"></a>Permission Management
Change Ownership

```shell
$ find / -user linda # search files owned by linda
$ find / -group ssh_users # find files owned by a group
$ chown user01:ssh_users test_file # change user and group ownership
$ chown -R user02:user02 testdir/ # recursively change ownership of directory and below
$ chgrp user01 testfile # change only group 
```

newgrp command

```shell
$ groups user01 # show primary group followed by secondary groups
	user01 : user01 wannabe_admins archusers
$ su - user01
$ newgrp archusers # open new shell with primary group archusers. (temporary)
$ groups
	archusers user01 wannabe_admins
$ touch LinuxMintIsBetter # create a file named after factual information
$ ll
	total 0
	-rw-r--r--. 1 user01 archusers 0 Jul 21 11:43 LinuxMintIsBetter # group changed
$ exit
$ groups
	user01 wannabe_admins archusers # changed back to normal
# user must be part of group specified or will be prompted for a password stored in gpasswd
```

Understanding Read, Write, and Execute Permissions

```shell
Permission        Applied to Files        Applied to Directories
----------        ----------------        ----------------------
Read             View file content        List contents of directory
Write        Change contents of a file    Create and delete files and subdirs
Execute         Run a program file        Change to the directory
```

Applying Permissions

```shell
$ chmod 740 somefile # absolute mode. (my preferred mode)
$ chmod u+x somefile # relative mode.
$ chmod +x somefile # omit "to whom" part to apply to all entries
$ chmod -R a+X files/ # change execute perms recursively for all subdirectories and not files
```

Understanding Advanced Permissions

```shell
# 4 set user ID (SUID)
# file will run as set user
$ ll /usr/bin/passwd
	-rwsr-xr-x. 1 root root 32648 Aug 10  2021 /usr/bin/passwd
# lowercase s means SUID and execute is set. Capital S means on SUID is set

# 2 set group ID (SGID)
# If set on an executable, gives user perms of group owner of that file when executed
# If set on a directory, all subdirectories and files obtain that set group ownership (more useful than above)
$ ll
	drwxr-sr-x. 2 user01 devops 30 Jul 21 13:27 dev_ops # SGID bit set
$ ll dev_ops/
	-rw-r--r--. 1 user01 devops 0 Jul 21 13:27 go_lang_proj.txt # created file owned by that group

# 1 sticky bit 
# when applied, can delete a file only if root, owner, owner of dir where file located
$ ls -ld /tmp
	drwxrwxrwt. 14 root root 4096 Jul 21 13:26 /tmp
```

Applying Advanced Permissions

```shell
$ chmod 4755 somedir/ # apply SGID to a directory
$ chmod u+s # SGID
$ chmod g+s # SGID
$ chmod +t # # sticky bit
$ chmod g+s,o+t data/sales/


Permission  On Files                   On Directories
---------   --------                   --------------

SUID        User executes file w/      No meaning
            perms of owner
            
SGID        User executes file w/      Files created in directory get the       
            perms of group owner       same group owner

Stickybit   No meaning                 Prevents users from deleting fies from 
                                       other users
```

Umask

```shell
# Files default permissions: 666
# Directory default permissions: 777
# umask 022 gives 644 for new files and 755 for new directories
Two ways to edit change defualt umask. useradd uses /etc/login.defs
1. add a umask.sh containing umask in /etc/profile.d/ directory 
2. edit .profile or .bashrc file for each specific user account
```

Attributes

```shell
$ man chattr
$ chattr +s somefile # set s attribute
$ chattr -s somefile # remove s attribute
$ lsattr # list attributes
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="networking"></a>Configuring Networking
addr and link

```shell
$ ip addr # configure and monitor network addresses
$ ip route # configure and monitor routing information
$ ip link # configure and monitor link state

# changes with ip command are not persistent after reboots
$ ip addr show # list network interfaces
$ ip -br addr show # better readability
$ ip -c a # -c=add color 
$ ip link show # show only link info
$ ip -s link show # shows rx and tx packet info
$ ip link set eth0 up # bring interface up or down
```

Routing

```shell
$ ip route show # show routing info. -c for color
# line 1 shows default route
# line 2 shows lines that identify local connected networks
```

Ports and Services

```shell
$ ss -lt # show listening tcp ports
$ ss -tulpn # tcp, udp, listen, process, and numeric
# ip4 loopback 127.0.0.1, ip6 loopback ::1
# listening for all ip4 addrs *, listening for all ip6 addrs :::*
```

nmtui and nmcli

```shell
# RHEL 9 is managed by the NetworkManager service
$ systemctl status NetworkManager # verify its status
# when this service comes up it reads network card configuration scripts located at
# /etc/NetworkManager/system-connections/
# these scripts have a name that starts with its network interface name
device - a network interface card
connection - configuration used on a device
# you can create multiple connections for a device
$ nmcli general permissions 
```

nmcli

```shell
$ man nmcli
$ man nmcli-examples
$ man nmcli | grep -A 15 ^EXAMPLES
$ man 5 nm-settings-nmcli
# tab auto-complete can be helpful. (bash-completion package)
$ nmcli <tab twice> 
$ nmcli connection <tab twice>

$ nmcli con show # show all connections
$ nmcli con show enp0s3 # show all properties of connection. long list
$ man 5 nm-settings # find out what these settings are doing
$ nmcli # overview of current configured devices and status
$ nmcli dev status # list all devices
$ nmcli dev show enp0s3 # settings for a specific device
```

Manage Network Connections: create a connection and manage its status

```shell
# run from console session, not SSH
# create network connection named dhcp, type ethernet, auto=dhcp:
$ nmcli con add con-name dhcp type ethernet ifname enp0s3 ipv4.method auto 
# create network connection named static. define static ip and gateway:
$ nmcli con add con-name static ifname enp0s3 autoconnect no type ethernet ipv4.method manual ip4 10.0.0.10/24 gw4 10.0.0.1
$ nmcli con show
$ nmcli con up static # switch to connection named static
$ nmcli con up dhcp # switch to connection named dhcp
$ nmcli con up enp0s3 # switch back to default
$ nmcli con mod # change current connection properties
# while adding a con use 'ip4' but while modifying existing con use 'ipv4'

# change autoconnect at reboot
$ nmcli con mod static connection.autoconnect yes
$ nmcli con mod dhcp connection.autoconnect no

$ nmcli -f name,autoconnect,autoconnect-priority,state c s # show con name and autoconnect property. -f=fields
$ nmcli -f <tab twice> # show possible fields
$ nmcli con add type <tab twice> # see all options
# connection.autoconnect-priority <default 0> low priority=-999 to 999=high priority

$ nmcli con mod enp0s3 ipv4.dns "8.8.8.8 1.1.1.1" # add multiple dns servers 
$ nmcli con show enp0s3 | grep ipv4.dns: # verify dns set serversnmc
```

Change Connection Parameters with nmcli

```shell
# while ADDING a con use 'ip4' but while MODifying existing con use 'ipv4'
$ nmcli con mod static connection.autoconnect no # make sure connection named static does not connect automatically
# or just 'nmcli con mod static autoconnect no'
$ nmcli con mod static ipv4.dns 10.0.0.10 # add/change dns server to connection
$ nmcli con mod static +ipv4.dns 8.8.8.8 # append second item w/ same params use +
$ nmcli con mod static ipv4.addresses 10.0.0.100/24 # change parameter of existing ip
$ nmcli con mod static +ipv4.addresses 10.20.30.40/16 # add second ip, use + sign
$ nmcli con up static # activate connection properties. down then up to reset

# You can modify an existing connection’s IP address and gateway using nmcli like this
$ nmcli con mod enp0s3 ipv4.addresses 192.168.1.50/24 ipv4.gateway 192.168.1.1 ipv4.method manual
```

nmtui

```shell
# 3 menu options of nmtui
Edit a connection # create new connections or edit existing
Activate a connection # use to re-activate a connection
Set system hostname # set hostname of computer
# after editing a connection you need to deactivate and activate it again!
$ nm-connection-editor # GUI
```

Hostname and Name Resolution

```shell
# 3 ways to change hostname
$ nmtui
$ hostnamectl set-hostname <hostname>
$ /etc/hostname

$ hostnamectl set-hostname rhelsver.rh.com # set hostname
$ hostnamectl status # check changes
$ cat /etc/hostname # check changes
```

/etc/hosts

```shell
# hostnames will be looked up first before dns is used in /etc/hosts if
# set in /etc/nsswitch.conf like this:
$ cat /etc/nsswitch | grep ^hosts:
	hosts: file dns myhostname
	# file: The system first checks /etc/hosts for a match. This is fast and local.
	# dns: If the hostname isn't found in /etc/hosts, it queries a DNS server (as configured in /etc/resolv.conf).
	# myhostname: If still unresolved, it checks the system's own hostname (like localhost, your-machine-name, etc.). This is provided by the systemd-hostnamed service if you’re using systemd.

$ cat /etc/hosts
	# <ip>  <fqdn> <shortname> 
	192.168.0.131	rhelsvr01.example.com rhelsvr01

# for actual dns you should set using tools like nmcli
$ nmcli con mod enp0s3 +ipv4.dns "1.1.1.1 8.8.8.8" # '+' if appending

# if your computer is configured to get the network config from a dhcp server, the dns server is also set via dhcp. if you do not want this to happen use this command
$ nmcli con mod enp0s3 ipv4.ignore-auto-dns yes
	# yes = ignore the DNS servers assigned via DHCP (don’t use them)
	# no (default) = accept and use the DNS servers assigned by DHCP

# After modifying the connection, restart it for the changes to take effect
$ nmcli con down enp0s3 && nmcli con up enp0s3

$ getent hosts rhelsvr01 # get entries from Name Service Switch libraries. nsswitch
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="dnf"></a>Managing Software
Register and Attach Subscriptions to a System (not needed for RHCSA??)

```shell
$ subscription-manager # tool to manage software from cli
$ subscription-manager register # will prompt for red hat creds to register
$ subscription-manager list --available # see what account is entitled to
$ subscription-manager attach --auto # registering is not enough. use to auto attach subscription to repos available
$ subscription-manager list --consumed # see what subscriptions currently using
$ subscription-manager unregister # unregister if you have a limited number of systems
$ /etc/pki/ # after registering and attaching a subscription, entitlement certificates are written here
$ dnf repolist # list enabled repos
$ dnf repolist --all # list disabled and enabled repos
```

Specifying Repo to Use (outdated - use config-manager)

```shell
# to tell your server which repo to use, create a '.repo' file in:
$ /etc/yum.repos.d/
# at a minimum the file needs these params:
[label] # .repo file can contain different repos. each section starting with a label that ids the specific repo
name= # specify the name of repo to use
baseurl= # contains the url that points to a specific location
# for RHCSA include 
gpgcheck=0 # will do gpg check without option
 
# Optional but commonly used:
enabled=1: # Enables the repo (defaults to 1 if not specified).
gpgcheck=1: # Enables GPG signature checking (recommended for security).
gpgkey=: # URL or path to the GPG key if gpgcheck=1.
```

Using config-manager 

```shell
# creates repo client file for you specifying the repo using http:
$ dnf config-manager --add-repo=http://myrepo.example.com/BaseOS
$ dnf config-manager --add-repo=file:///repo/BaseOS # adds the local test repo
gpgcheck=0 # add to newly created .repo file to disable gpg checking
$ dnf repolist # to show new repo enabled
# use http://, ftp://, or file://
$ dnf repository-packages <repoID> list
```

Using DNF

```shell
$ dnf search <packageName> # search packages for a string that occurs in the package name or summary
$ dnf search all <packageName> # search packages for string that occurs in the package name, summary, or description
$ dnf provides */createrepo # perform deep search in the package to look for. Will show name of package
$ dnf info <packageName> # provide more info on a package
$ dnf clean all # remove all stored metedata
$ dnf [install|remove] <packageName> # install or remove a package
$ dnf list [all|installed] # list all or installed packages
$ dnf list kernel # show current and available kernel if any
$ dnf upgrade <packageName> # upgrade all packages or specified package

$ dnf group list # list package group
$ dnf group install <groupName> # install group packages
$ dnf group list hidden # list subgroups
$ dnf group info "Container Management" # to see what is in a group
```

Managing Package Modules

```shell
$ man dnf.modularity
$ dnf module list <moduleName> # list all available modules in enabled repos or list specific streams
$ dnf module info [php|php:8.1] # see info on a stream or stream version
$ dnf module enable php:8.3/common # dnf module enable <name>:<stream>/<profile>
$ dnf module list php # php  8.3 [e]. e means enabled
$ dnf module list --enabled 
$ dnf module reset php
$ dnf module list php # php  8.3 [d] # d means disabled
```

RPM

```shell
$ rpm -qa | less # search all installed packages. /openssh for faster search
$ rpm -ql openssh-server # list files included in package and configs
$ rpm -qd openssh-server # show all documentation for a package
$ rpm -qc openssh-server # show all config files for package
$ rpm -qf /bin/ls # find the name of the package ls comes from
$ sudo rpm -i ./package.rpm # install specified package but not dependencies
```

Creating a repo with a mounted ISO file in a VM

```shell
# attach the ISO directly to the VM as if it were a DVD:
1. Go to Settings > Storage.
2. Under the IDE or SATA controller, click the empty disk icon.
3. Click the CD icon on the right → Choose a disk file → select your ISO (e.g., rhel-9.iso).
4. Start the VM.
$ umount /dev/sr0 # this is where the ISO automounts to. use lsblk
$ mkdir /repo # create mount point 
$ lsblk -f # grab UUID
$ vi /etc/fstab 
	UUID=<uuid> /repo iso9660 defaults 0 0 # add to fstab for reboot persistance
$ mount -a # mount all in fstab. will list any errors
$ mount | grep sr0 # check if mounted
$ ls /repo # will list BaseOS and Appstream which are the repos
$ dnf config-manager --add-repo="file:///repo/BaseOS" # create .repo file
$ dnf config-manager --add-repo="file:///repo/AppStream"
$ dnf repolist # list repos. There were none prior
$ gpgcheck=0 # go into the new .repo files and add this
# Now you can install some packages
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="procs"></a>Managing Processes
Managing Jobs

```shell
$ & # background a job if appended to commands
$ Ctrl+Z # Stops job temporarily
$ Ctrl+D # Sends EOF character to job to indicate stop waiting for input
$ Ctrl+C # Cancel cuurent interactive job
$ bg # background job after using Ctrl+Z
$ fg # foreground job
$ jobs # list job id and use fg 1 to foreground
$ top # type k, then PID, to kill a process
```

Managing Process Priorities

```shell
$ nice # change start process priority
$ renice # change active process priority
$ top # type r, enter PID to change nice value

$ nice -n 5 if=/dev/zero if=/dev/null & 
$ ps aux | grep dd # list pid
$ renice -n 10 -p 1234 # lower prioority by pid
$ top # shows dd at the top. type k to kill 
```

Sending Signals

```shell
$ man 7 signal # overview of signals
$ SIGTERM (15) # asks to stop a process
$ SIGKILL (9) # force a process to stop
$ SIGHUP (1) # hang up process. proccess will reread its configuration file
$ kill <pid> # kill command sends signals to process, default SIGTERM 15
$ kill -9 <pid> # send SIGKILL to a process
$ kill -l # list available signals for kill
$ ps fax | grep -B5 dd # shows hierarchy of dd process and ppid that started it
$ killall <processname> # to kill all processes by name

$ ps aux | grep defunct # list zombie procs
$ kill -SIGCHLD <PPID> # tell parent proc to rem child procs. systemd will adopt and then removed after a few seconds
$ kill -9 <pid> # if above fails
```

top

```shell
$ top # start top, view load avg and processes
$ R # running
$ S # sleeping
$ Z # zombie
$ k # enter pid to kill from within top
$ r # edit nice value or process
# load avg should not be higher than number of cores. 'lscpu' to check
```

tuned

```shell
$ dnf install -y tuned
$ systemctl enable --now tuned # enable and start 
$ tuned-adm active # check current active profile
$ tuned-adm list # view profiles available
$ tuned-adm recommend # gives a recommendation
$ tuned-adm profile virtual-guest # switch active to virtual-guest
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="systemd"></a>Working with Systemd
Units

```shell
$ systemctl -t help # list unit types
$ #Unit files are found in these locations:
$ /run/systemd/system # hightest order of precedence. Contains unit files that have been generated automatically. overwrites settings set elsewhere
$ /etc/systemd/system # second order of precedence. contains custom unit files
$ /usr/lib/systemd/system # third order of precedence. contains default unit files from installed packages. Do not edit directly
$ systemd.unit(5) and systemd.service(5) man pages
```

Service Unit (*.service files)

```shell
# used to start any type of service. you can start any process, daemon processes or commands
# service files contain 3 sections. Unit, Service, and Install:

# the first part describes unit and defines dependencies
# After statement, which indicates that this unit should start after the listed unit
# the Before indicates that this should start before the listed unit
[Unit]
Description=OpenBSD Secure Shell server
After=network.target 
[Service]
Type=notify
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
# describes how to start and stop a service and request installation
# type option is used to specify how the process should start
[Install]
WantedBy=multi-user.target
# indicates in which target this unit has to be started
$ man 5 systemd.service # more details
```

Mount Units (*.mount files)

```shell
# mount units specify how a file system should be mounted on a specific directory
# mount units are an alternative for mounting file systems through /etc/fstab
[Unit]
[Mount]
What=tmpfs
Where=/tmp
Type=tmpfs
Options=nosuid,nodev,noexec
[Install]
```

Socket Units (*.socket files)

```shell
# a socket creates a method for applications to communicate
# a socket can be defined as a port or file that systemd will listen on
# the service doesnt have to run unless a connection comes to the socket
# every socket NEEDS a corresponding service file
[Unit]
Wants=cockpit-issue.service # the wanted service file
[Socket]
ListenStream=9090
[Install]
WantedBy=sockets.target

$ ListenStream= # defines the TCP port systemd should listen on
$ ListenDatagram= # for UDP
```

Target Units (*.target files)

```shell
# a target unit is a group of units
# targets can have dependencies on other targets
# the dependencies are defined in a target unit
[Unit]
$ systemctl list-dependencies 
# target files don't contain info on units that should be included
# that is defined in the [Install] section of different unit files
# when using 'systemctl enable', the [Install] section of that unit is considered to determine which target should be added
# when you add a unit to a target a symbolic link is created under the hood in the target dir in /etc/systemd/system
# if you enable vstpd service, a symbolic link will be added in /etc/systemd/system/multi-user.target.wants/vstpd.service 
# and it will point to the unit in /usr/lib/systemd/system/vstpd.service to automatically start
# to systemd a 'want' a symbolic link is known as a want
```

Managing Units Through Systemd

```shell
$ dnf install -y vsftpd
$ systemctl status vsftpd # verify status. rhel disables at install unlike ubuntu
$ systemctl start vsftpd # start service only until reboot
$ systemctl enable vsftpd # start automatically enabled. output shows sym-link
$ systemctl enable --now vsftpd # start automatically and start now 
```

systemctl unit commands

```shell
$ systemctl -t service # show only service units
$ systemctl list-units -t service # show all active service units(same above)
$ systemctl list-units -t service --all # show alos inactive service units
$ systemctl --failed -t service # show all failed services
$ systemctl status -l sshd.service # show detailed info about services
```

Managing Dependencies

```shell
# 2 ways to manage dependencies
# 1. unit types socket, timer, path are directly related to a service unit
# systemd makes a connection because the first part of the name matches(cockpit.socket, cockpit.service)
# accessing either of them will trigger the service unit
# 2. dependencies can be defined in the unit with Requires, Before, After, and Requisite
$ systemctl list-dependencies <unitName> # list dependencies
$ systemctl list-dependencies <unitName> --reverse # list Required units for this unit to be started
# ensure accurate dependency management w/ keywords in [Unit] section:
Requires # if this unit loads, units here will be loaded. if one is deactivated, this unit will deactivate
Requisite # if unit here is not loaded already, this unit will fail
Wants # the unit wants to load the units here, but will not fail if any listed units fail
Before # this unit will start before unit specified here
After # this unit will start after the unit specified here
```

Managing Unit Options

```shell
$ systemctl show sshd # show options available for specific unit. sshd.service unit in this example
$ systemctl edit # edit unit files in /etc/systemd/system, where custom unit files are created
# systemctl edit command creates a subdirectory of the service specified. example:
$ systemctl edit httpd.service # creates /etc/systemd/system/httpd.service.d/ and a file named override.conf is created
# all settings applied in this file over-write any existing settings in /usr/lib/systemd/system

$ dnf install -y httpd # install Apache2
$ systemctl cat httpd.service # show current config of unit file that starts apache
$ systemctl show httpd.service # print available config options for unit file
$ echo 'export SYSTEMD_EDITOR="/bin/vim"' >> .bashrc # set vim to systemd editor
$ systemctl edit httpd.service # add this to the override file:
	[Service] # add the section to add to 
	Restart=Always # Ensures httpd will restart after a failure
	RestartSec=5s #  Wait 10 seconds before restarting
$ systemctl daemon-reload # re-reads the systemd unit files
$ systemctl start httpd # start service 
$ systemctl status httpd # check status
$ pkill httpd # kill the service
$ systemctl status httpd # will be stopped but will restart in 5 seconds
$ sudo systemctl revert httpd.service # This deletes the drop-in override and reverts to the default unit file.
```

Man Pages

```shell
$ man systemd.unit # general unit directives
$ man systemd.service # service-specific directives
$ man systemd.mount # for .mount units
$ man systemd.socket # for .socket units
$ man systemd.directives # list directives
```

\[Unit\] Purpose: Defines metadata and dependencies.

```shell
$ Description= # Human-readable summary of the unit
$ After= # Start this unit after another
$ Requires= # Hard dependency — starts listed unit too
$ Wants= # Soft dependency — starts if available
$ Conflicts= # Prevents this unit from running with another
$ ConditionXYZ= # Conditional checks (e.g., ConditionPathExists=)
```

\[Service\] Purpose: Used in .service unit files — defines how to start and manage the service.

```shell
$ ExecStart= # Command to start the service
$ ExecStop= # Command to stop it
$ ExecReload= # Command to reload it
$ Restart= # Restart policy on failure
$ RestartSec= # Time to wait before restarting
$ User= / Group= # Runs the service as this user or group
$ Environment= # Set environment variables
$ Type= # Tells systemd how to track the service (e.g., simple, forking)
```

\[Install\] Purpose: Defines how the unit integrates into systemctl enable and startup targets.

```shell
$ WantedBy= # Makes this unit start when the listed target is reached (e.g., multi-user.target)
$ RequiredBy= # Like WantedBy, but a hard dependency
$ Alias= # Creates symlink aliases for the unit
```

\[Socket\] Used in .socket units — defines how to listen for incoming connections.

```shell
$ ListenStream= # Listen on a TCP port or Unix stream socket
$ ListenDatagram= # Listen on a datagram socket (UDP-like)
$ Accept= # Whether to spawn a service per connection (yes/no)
$ Service= # Which .service unit to activate on connection
$ SocketUser= # User to own the socket
$ SocketGroup= # Group to own the socket
```

\[Mount\] Used in .mount units — defines mount points like /etc/fstab.

```shell
$ What= # Source device or filesystem
$ Where= # Mount point directory
$ Type= # Filesystem type (e.g., ext4, nfs)
$ Options= # Mount options (noatime, ro, etc.)
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="tasks"></a>Scheduling Tasks
Systemd Timers (*.timer files)

```shell
# 'systemd timers' are the default solution for scheduling tasks
# 'cron' is legacy and 'at' is for sheduling occasional jobs

# Timers are always used with Service files, and the names should match
# the 'Service' unit defines how it should start, 'Timer' unit defines when.
# if you need a Service to be started by a Timer, you enable the Timer, not the Service
$ systemctl cat logrotate.timer # also accompanies logrotate.service
# to define how the timer should be started use, [Timer]
[Timer]
OnCalendar=daily # describes when timer executes. current is daily
AccuracySec=1h # time window within it should execute
Persistent=true # must have OnCalendar set. takes boolean logic. it set to true, it stores the last time it was triggered on disk to trigger when needed
# Important options:
OnActiveSec= # defines a timer relative to the moment the timer is activated
OnBootSec= # defines a timer relative to when the machine was booted
OnStartupSec= # specifies a time relative to when the service manager was started. 
OnCalender= # defines timers on calender event expressions like 'daily'. see 'man systemd.time'
$ man systemd.time

$ systemctl list-units -t timer # list all timers
$ systemctl list-unit-files logrotate.* # lists .service and .timer
$ systemctl cat logrotate.service # there is no [Install] section
$ systemctl status logrotate.service # shows 'TriggeredBy: logrotate.timer'
$ dnf instal -y sysstat 
$ systemctl list-unit-files sysstat*.* # show unit files added from package
$ systemctl cat sysstat-collection.timer # shows OnCalendar=*:00/10 which ensures that it will run every 10 minutes
```

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

## <a name="summary"></a>Summary

Enter summary here: Still in progress...

[Back to Top](https://github.com/HunterCartier702/RHCSA-Home-Lab/blob/main/README.md#intro)

