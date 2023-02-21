# Raspberry Toolset

This repository contains all the software that I have running in my Raspberry.   
It is an extended/modified fork of [this repo](https://github.com/pablokbs/plex-rpi). Kudos to my man Pelado :)

The Raspberry I am using is: [Raspberry Pi 4 Model B (8GB)](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/).

## Available applications

### [Jellyfin](https://jellyfin.org/)
Jellyfin enables you to collect, manage, and stream your media.

### [Flexget](https://flexget.com/)
FlexGet is a multipurpose automation tool for all of your media.

### [Transmission](https://transmissionbt.com/)
A Fast, Easy and Free Bittorrent Client.

### [Samba](https://www.samba.org/)
Samba is a free software re-implementation of the SMB networking protocol for Windows clients.

### [Pi-hole](https://pi-hole.net/)
The Pi-hole® is a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

### [Heimdall](https://jellyfin.org/)
Heimdall Application Dashboard is a dashboard for all your web applications. It doesn't need to be limited to applications though, you can add links to anything you like. 

### [Portainer](https://www.portainer.io/)
Portainer is a powerful, open source toolset that allows you to easily build and manage containers in Docker, Docker Swarm, Kubernetes and Azure ACI.  

## Initial requirements

Add your user (change `xavo` with your username).

```
sudo useradd xavo -G sudo
```

Add this to `/etc/sudoers` to run sudo without password:

```
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

Add this line to `/etc/ssh/sshd_config` so only your user can use ssh.

```
echo "AllowUsers xavo" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl enable ssh && sudo systemctl start ssh
```

Install basic packages:

```
sudo apt-get update && sudo apt-get install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common \
     vim \
     fail2ban \
     ntfs-3g
```

Install Docker:

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
echo "deb [arch=armhf] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update && sudo apt-get install -y --no-install-recommends docker-ce docker-compose
```

Change your Docker config so the temps are stored in the external disk.
Assuming the external device is mounted on: `/mnt/storage/`

```
sudo vim /etc/default/docker

# Add / Modify this line
export DOCKER_TMPDIR="/mnt/storage/docker-tmp"
```

Add your user to Docker group: 

```
# Add xavo to docker group
sudo usermod -a -G docker xavo
```

Mount the disk:

```
sudo su

# Search the disk we want to mount. (Example: /dev/sda)
fdisk -l

# Check if it is already mounted 
lsblk

# If the MOUNTPOINT header is mounted (Example: in /mnt), run:
umount /dev/sda1 /mnt

# Find the UUID of the disk partition
ls -l /dev/disk/by-uuid/

# Add the mount settings into the '/etc/fstab' file so when the machine is rebooted, it won't change.
# If the file system of your disk is ntfs, in the FILE_SYSTEM argument you'll have to install ntfs-3g and add "ntfs-3g" as label.
echo UUID="{UUID_OF_THE_DISK}" {NEW_DIRECTORY_TO_MOUNT} {FILE_SYSTEM (Example: ext4)} defaults,auto 0 0 | \
     sudo tee -a /etc/fstab

# Read from fstab and mount the disk. (Restarting the machine also works)
mount -a 
```

## Set up
Download this repo, create the env file and modify it. The docker compose reads from `.env`:

`cp .env_example .env`   

### Flexget
Flexget needs rights to write, so additional permissions have to be set up:   

`sudo chmod 777 {Mountpoint}/storage/movies`  
`sudo chmod 777 {Mountpoint}/storage/series`


### Pi-Hole
The best way to make run it is changing the DNS of the LAN in your router to point to the raspberry private IP.   
If that's not possible because router's firmware is trash, it has to be done client by client.   

[Guide](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245)

### Heimdall

The current version (2.5.5) has an [open issue](https://github.com/linuxserver/Heimdall/issues/840) that blocks the application after a while with errors 500.   
To avoid this, run this from project's root folder:

`mkdir /heimdall/config/www/`   
`cp .env_heimdall /heimdall/config/www/.env`

It is necessary to add the applications manually. This is quick and straightforward. You need to generate API auth keys from some applications (Jellyfin, Pihole, Transmission...) to get live data in the dashboard.

## Running it

Run docker:

`docker-compose up -d`

## Using it
Let's assume the Raspberry private address is `192.168.1.23` and we are trying to access it from the LAN.   

#### Heimdall: http://192.168.1.23:2550   
#### Portainer: https://192.168.1.23:9443   
#### Jellyfin: http://192.168.1.23:8096   
#### Transmission: http://192.168.1.23:9091    
#### Flexget UI: http://192.168.1.23:5050    
#### Pihole UI: http://192.168.1.23/admin    
#### Samba: Open the file explorer in Windows, press CTRL+L and type `\\192.168.1.23` (It is necessary to previously activate the SMB protocol in Windows)
