# Raspberry Toolset

This repository contains all the software that I have running in my Raspberry.
It is an extended/modified fork of [this repo](https://github.com/pablokbs/plex-rpi). Kudos to my man Pelado :)

The Raspberry I am using is: [Raspberry Pi 4 Model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/).

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

# If the MOUNTPOINT header is blank, we have to mount it. We will mount it at /mnt in this example
mount /dev/sda1 /mnt
```

## Set up

Download this repo, create the env file and modify it. The docker compose reads from `.env`:

`cp .env_example .env`

### Pi-Hole
The best way to make it run is changing the DNS of the LAN in your router to point to the raspberry private IP.   
If that's not possible because Router's firmware is trash, it has to be done client per client.   

[How to](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245)

## Running it

Run docker:

`docker-compose up -d`

## Using it
Let's assume the Raspberry private address is `192.168.1.23` and we are trying to access it from the LAN.

To access Jellyfin: http://192.168.1.23:8096   
To access Transmission: http://192.168.1.23:9091    
To access Flexget UI: http://192.168.1.23:5050    
To access Pihole UI: http://192.168.1.23/admin    
 