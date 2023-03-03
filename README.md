# Raspberry Toolset

This repository contains all the software that I have running in my Raspberry.   
It is an extended/modified fork of [this repo](https://github.com/pablokbs/plex-rpi). Kudos to my man Pelado :)

The Raspberry I am using is: [Raspberry Pi 4 Model B (8GB)](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/).

## Available applications from the Docker compose file

### [Jellyfin](https://jellyfin.org/)
Jellyfin enables you to collect, manage, and stream your media.     

### [Jackett](https://github.com/Jackett/Jackett)
Jackett works as a proxy server: it translates queries into tracker-site-specific http queries.     

### [Sonarr](https://sonarr.tv/)
Sonarr is a PVR for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will grab, sort and rename them.

### [Radarr](https://radarr.video/)
Radarr is a movie collection manager for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new movies and will interface with clients and indexers to grab, sort, and rename them.

### [Qbittorrent](https://www.qbittorrent.org/)
A Fast, Easy and Free Bittorrent Client.     

### [Samba](https://www.samba.org/)
Samba is a free software re-implementation of the SMB networking protocol for Windows clients.

### [Pi-hole](https://pi-hole.net/)
The Pi-hole® is a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

### [Portainer](https://www.portainer.io/)
Portainer is a powerful, open source toolset that allows you to easily build and manage containers in Docker, Docker Swarm, Kubernetes and Azure ACI.  

### [Uptime Kuma](https://uptime.kuma.pet/)    
A self-hosted monitoring tool.    

### [Rsnapshot](https://rsnapshot.org/)    
Rsnapshot is a filesystem snapshot utility based on rsync. 

## Initial requirements

Add your user (change `xavo` for your username).

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

### Pi-Hole
The best way to make run it is changing the DNS of the LAN in your router to point to the raspberry private IP.   
If that's not possible because router's firmware is trash, it has to be done client by client.   

[Guide](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245)

### Sonarr
Sonarr docker image is not working for arm64 arch, so a manual installation has to be done following the [official docs](https://sonarr.tv/#downloads-v3-linux).    

## Running it

Run docker:

`docker-compose up -d`

## Using it
Let's assume the Raspberry private address is `192.168.1.23` and we are trying to access it from the LAN.   

#### Jellyfin: http://192.168.1.23:8096   
#### Qbittorrent: http://192.168.1.23:8088    
#### Portainer: https://192.168.1.23:9443   
#### Jackett: https://192.168.1.23:9117   
#### Radarr: http://192.168.1.23:7878    
#### Sonarr: http://192.168.1.23:8989    
#### Pihole UI: http://192.168.1.23/admin    
#### Uptime Kuma: http://192.168.1.23:3001    
#### Samba: Open the file explorer in Windows, press CTRL+L and type `\\192.168.1.23` (It is necessary to previously activate the SMB protocol in Windows)
