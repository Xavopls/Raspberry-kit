# My Raspberry Toolset

This repository contains all the software that I have running in my Raspberry.
It is an extended/modified fork of [this repo](https://github.com/pablokbs/plex-rpi). Kudos to my man Pelado :)

The Raspberry I am using is: [Raspberry Pi 4 Model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/).

With this repo you'll be able to create a server that downloads your tv shows and movies automatically, and when finished, these will be accessible from Jellyfin.

## Available applications

### [Jellyfin](https://jellyfin.org/)
Jellyfin enables you to collect, manage, and stream your media.

### [Flexget](https://flexget.com/)
FlexGet is a multipurpose automation tool for all of your media.

### [Transmission](https://transmissionbt.com/)
A Fast, Easy and Free Bittorrent Client.

### [Samba](https://www.samba.org/)
Samba is a free software re-implementation of the SMB networking protocol for Windows clients.


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

Mount the disk. If your disk is NTFS it is necessary to add ntfs-3g:

```
sudo su

# Search the disk we want to mount. As example we will be using the partition sdb1)
fdisk -l

# With this command we can get the UUID
ls -l /dev/disk/by-uuid/

# Mount the disk in the file /etc/fstab 
echo UUID="{UUID of disk}" {Path to be mounted on (for example: /mnt/storage)} ntfs-3g defaults,auto 0 0 | \
     sudo tee -a /etc/fstab
# To update and read fstab:
mount -a 
```

## Running it

Download this repo, create the env file and modify it. The docker compose reads from `.env`:

`cp .env_example .env`

Run docker:

`docker-compose up -d`

## Using it
Let's assume the Raspberry private address is `192.168.1.23` and we are trying to access it from inside the LAN.

To access Jellyfin: http://192.168.1.23:8096   
To access Transmission: http://192.168.1.23:9091    
To access Flexget UI: http://192.168.1.23:5050    
 