---
title: "A good debian 12 server install with docker and zsh!"
date: 2024-04-02
draft: false
description: "OwO"
tags: ["hyprland", "workflow"]
---

How to make a good Debian 12 server install with Docker and all the tools I use for such a server, like eza and zsh.

# A good debian 12 server install with docker and zsh!

## Prerequisites 
We first need to get the ISO. I'm going to go with Debian 12 Bookworm, and you can
of course, install this on hardware, but I'm going to install it on a VM.

## Installation

### VM
I will use a VM to setup the Debian setup. You can of course install this.
on hardware if you want, but for ease of installation, I will use a VM and
The software I use to do that is QEMU/KVM.

- **Firmware:** UEFI
- **Cpu:** 1 Socket, 6 Cores and 1 Thread.
- **Ram:** 8192MiB
- **Gpu:** virtio(2d)
- **Hdd:** 100GiB

### Debian Installer
I will walk you through the installer from the domain,
to manually partition disks.

#### Domain 
You now need to enter your hostname, like debian-server or something like that, and for 
domain normally you can skip this, but if you have setup pfsense or opnsense than 
You can enter the domain after the first dot, so for me, that would be home.arpa (the default).

#### Partitioning 
You need to choose guided remove swap because we are going to use zram, then remove root and partition it with xfs or btrfs. if using an SSD If not, you can use ext4. It will give a warning after continuing because there is no swap.
but you can just ignore that by hitting no and then continue with the install.

#### Mirrors
You should choose the default (deb.debian.org) if you don't know which to choose.

#### Desktop Selection
Untick all but 'Debian desktop environment', 'standard utils' and enable 'SSH server'.

#### Finish!
It should now be installed.

### Setting up

#### Sudo
We now need to go to tty2 (ctrl + alt + f2), then login with root because we need to add our user to the sudoers group.
and we do that with: 
```bash
/usr/sbin/usermod -aG sudo user
```
then we exit root (ctrl + d) and login with our user, and we should now be in the sudoers file!

#### SSH
Now we need to setup an SSH connection. We do that by first enabling the service:
```
sudo systemctl enable --now ssh
```
We need to check for what IP to connect to with ip a:
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:09:75:ef brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.221/24 brd 192.168.122.255 scope global dynamic noprefixroute enp1s0
       valid_lft 2798sec preferred_lft 2798sec
    inet6 fe80::5054:ff:fe09:75ef/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
And in 2. inet 192.168.122.221 is the IP we need to connect to.

#### Refreshing Mirrors (and fixing kitty)
We now need to fix Kitty because, as you may have noticed, the SSH connection is acting up.
This is Kitty to fix that we need to do this:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install kitty -y
sudo apt remove gdm3 -y
```
And then reconnect with the SSH session.

#### Installing Required Packages
We will now install all the required packages for this server installation:
```bash
sudo apt install cargo zram-tools fuse-overlayfs slirp4netns neovim git curl zsh neofetch make cmake rustc btop uidmap dbus-user-session -y
```

For Mcfly:
```bash
curl -LSfs https://raw.githubusercontent.com/cantino/mcfly/master/ci/install.sh | sudo sh -s -- --git cantino/mcfly
```

For eza:
```bash
sudo mkdir -p /etc/apt/keyrings
wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | sudo gpg --dearmor -o /etc/apt/keyrings/gierens.gpg
echo "deb [signed-by=/etc/apt/keyrings/gierens.gpg] http://deb.gierens.de stable main" | sudo tee /etc/apt/sources.list.d/gierens.list
sudo chmod 644 /etc/apt/keyrings/gierens.gpg /etc/apt/sources.list.d/gierens.list
sudo apt update && sudo apt install -y eza
```

And for dust:
```bash
curl -LSfs "https://github.com/bootandy/dust/releases/download/v1.0.0/du-dust_1.0.0-1_amd64.deb" -o dust.deb
sudo dpkg -i dust.deb && rm dust.deb 
```
#### Zram for swap
To set up zram, we just need to add these lines to the config and start the service for zram:
```bash
sudo /bin/su -c "echo -e "PERCENT=60" | sudo tee -a /etc/default/zramswap"
sudo /bin/su -c "echo -e "ALGO=zstd" | sudo tee -a /etc/default/zramswap"
sudo zramswap start 
```

#### Setting up zsh
Just git clone this repo and execute the script, it will install and setup zsh:
```bash
git clone https://git.kaleyfischer.xyz/DRAGONTOS/zsh-dotfiles.git && zsh-dotfiles
chmod +x install.bash
bash install.bash
sudo reboot
```

### Setting up docker with a website!
We are now going to setup Docker with a [website](https://git.kaleyfischer.xyz/DRAGONTOS/website)!

#### Docker Install 
We need to add some lines to /etc/sysctl.conf:
```bash
sudo /bin/su -c "echo 'net.ipv4.ip_unprivileged_port_start=0' >> /etc/sysctl.conf"
sudo /bin/su -c "echo 'kernel.unprivileged_userns_clone=1' >> /etc/sysctl.conf"
sudo /bin/su -c "echo 'vm.max_map_count=262144' >> /etc/sysctl.conf"
sudo sysctl --system
```
It's now time to install Docker!
We first need to add the Docker keyrings, and you can do that with this:
```bash
sudo apt-get update
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
After that is done, we need to add the repo to our sources:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```
We can now finally install Docker with the most up-to-date versions:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
Now that Docker is installed, we need to test it first to check if it is installed.
correctly or not, and you can do that with this:
```bash
sudo docker run hello-world
```
If it worked, then Docker is installed correctly!

#### Setting up rootless for docker
Now that we have Docker installed, we don't want to run everything with root and
want to run it securely with our user in rootless mode. To do that, we need to run
this simple script from Docker themselves:
```bash
sudo systemctl disable --now docker.service docker.socket
dockerd-rootless-setuptool.sh install
systemctl --user enable --now docker
```
And again, to check if it's installed correctly, we can run this command:
```bash
docker run hello-world
```

#### Setting up nginx
Now that Docker is installed and working, we now need to add some folders for them.
where to place the containers and such, and to do that, we just need to add these:
```bash
mkdir ~/docker && cd ~/docker
```

For setting up a site with nginx, just clone my git repo for a Docker container with nginx:
```bash
git clone https://git.kaleyfischer.xyz/DRAGONTOS/nginx-docker && cd nginx-docker
docker compose up -d
```

## Wrapping It Up 
I hope that you now have a working Debian 12 server installed with Docker and a running Nginx site!
And for help, you could always DM me on Twitter for the time being until I have my own Masadon account.

