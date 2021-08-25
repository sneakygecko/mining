# Crypto Mining on Ubuntu 20.04 LTS 
This will show you how to setup a mining rig with an NVIDIA RTX 3070 to mine Ravencoin (RVN), Ergo (ERGO), Ethereum (ETH)(pre Ethereum 2.0) or Ethereum Classic (ETC).
Any Nvidia GPU will have similar settings.
You will be able to run this miner headless if you desire.

## Install Ubuntu Desktop 20.04.X
- Download the latest LTS release from https://ubuntu.com/download/desktop
- Make a bootable USB with the iso
  - Use Rufus on Windows
  - Use DD on linux & Mac
- Plug in your Live USB & Turn on your computer.
- For Dells keep pressing F12 at the logo screen to get to you boot devices.
- Select the USB as the Boot Device
- Do a minimal install
  - I installed it onto a USB drive not the actual hard drive. If you do this change you BIOS settings boot order to boot from the USB first whenever it is plugged in. 
  - Enable Auto Login.
  - Enable download proprietary drivers (these are your NVIDIA drivers).
  - I set my User name as "user".
  - I set my hostname as "computer".
  - Use a secure password/ passphrase i.e. "deplored nutcase unmoving uptake payment", brought to you by BitWarden.
  - Don't encrypt the hard drive otherwise Auto Login won't work.

## Set up Ubuntu
- Startup your freshly installed Ubuntu.
- If using wifi connect to your network of choice now.
- Once Connected we want to set a static IP (This way we can connect to it later using SSH)
- Click you network, then settings, then go to IPv4, select Manual
  - My home network is on 192.168.1.1/24
  - Address: 192.168.1.3
  - Netmask: 255.255.255.0
  - Gateway: 192.168.1.1
  - DNS: 1.1.1.1 
  - Disable IPv6
- Turn off wifi, turn it back on to get your new IP
- Open a terminal. Don't worry it'll be fine. It's not that scary :) 
- Copy the lines into the terminal and enter in your password if required.
```bash
sudo gedit /etc/apt/apt.conf.d/50unattended-upgrades
```
- We want the computer to automatically install any security patches and restart if necessary. Don't want to get hacked and be mining for someone else.
- Find, change and remove the # (Uncomment) from the following lines
```
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
```
- Update Ubuntu
```bash
sudo apt update
sudo apt upgrade
```
- Install some stuff
```bash
sudo apt install tmux nvidia-cuda-toolkit
```
- If you want to use SSH, install and set firewall
```bash
sudo apt install openssh-server
sudo ufw allow ssh
```

sudo nvidia-xconfig --enable-all-gpus --cool-bits=28
sudo reboot now
```

- Download the latest T-Rex Miner (t-rex-0.21.6-linux.tar.gz) from https://github.com/trexminer/T-Rex/releases
