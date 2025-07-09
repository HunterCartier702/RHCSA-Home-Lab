# RHCSA-Home-Lab

## Table of Contents

  - [Introduction](#intro)
  - [Inital Setup](#initial)
  - [Installing Cockpit](#cockpit)
  - [Summary](#summary)

## <a name="intro"></a>Introduction 
I decided to try my luck at the RHCSA exam. I decided to use a mix of VM’s on Virtual Box and physical hardware to study for the Red Hat certification. I will be documenting some of what I learn while doing this lab in hopes that it helps reinforce the knowledge. By copying down the commands I run and looking them over I know it will help me retain the material. For this project I bought a refurbished Lenovo for $120 dollars with these specs:

Lenovo ThinkCentre M910S Intel i7 3.20 GHz, 16GB DDR4 RAM, 1TB SSD, and I installed an extra 256GB SSD.

<p align="center"><img alt="Tower" src="rhel_server/00Tower.jpeg" height="auto" width="800"></p>
<p align="center"><img alt="HTOP" src="rhel_server/01HTOP.png" height="auto" width="800"></p>
<p align="center"><img alt="lscpu" src="rhel_server/03cpu.png" height="auto" width="800"></p>
<p align="center"><img alt="memory" src="rhel_server/02memory.png" height="auto" width="800"></p>
<p align="center"><img alt="SSD" src="rhel_server/04SSDInstall.jpeg" height="auto" width="800"></p>

**Side note:**

As a mainly Ubuntu user, I noticed some things right away. The package managers yum and dnf update the local cache automatically when installing packages. In Ubuntu you would run ‘apt update’ then ‘apt install <package>’ to update the local cache to the latest and then install. With dnf you can run ‘dnf install <package>’ and it updates and installs packages automatically. Yum is sym linked to dnf anyway and so I will use dnf. Also, the /bin directory is sym-linked to /usr/bin along with a few other directories being linked to a /usr/* directory. 

## <a name="initial"></a>Initial Setup
I ran through the anaconda installer which is really nice. I went with the “minimal install” software option. Openssh-server is installed by default, so I will transfer over the public key from my desktop and laptop to the authorized_keys  file for easier access. I will be using SSH for everything after the initial setup. 

**First commands:**

```shell
# update system since its a fresh install
$ sudo dnf upgrade -y
# install tailscale to access server remotely
$ curl -fsSL https://tailscale.com/install.sh | sh
$ sudo tailscale up
$ sudo dnf install vim -y
# from main desktop to server
$ scp .vimrc cartier@rhelsvr01:
# cp .vimrc with personal preferences so new users also get file upon creation
$ sudo cp .vimrc /etc/skel/
```

## <a name="cockpit"></a>Installing Cockpit
Cockpit is a set of tools that allows for web management of the system. It is hosted at https://localhost:9090 after installing. You can access the terminal from the browser, view logs, update software, and view storage just to name a few.

```shell
$ sudo dnf install cockpit
# to start service now and enable persistence after reboots
$ sudo systemctl enable --now cockpit.socket
```

<p align="center"><img alt="cockpit" src="rhel_server/05Cockpit.png" height="auto" width="800"></p>

## <a name="summary"></a>Summary

I turned an old gaming laptop into a home server running Ubuntu 24.04.2 LTS just to mess around and learn more about Linux. I’ve been using Linux for over a year now and wanted to put what I’ve learned into action. In this project, I set up Samba, NFS, MariaDB, Apache2 (with a custom web page), and even installed a new SSD just for the fun of partitioning and formatting it. Then I tried out virtualization which required a lot more setup than VirtualBox. Everything’s still a work in progress, but it’s been a great learning experience so far. More to come!

