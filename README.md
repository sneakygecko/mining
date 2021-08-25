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
- Now you can SSH from a different computer to this one using `ssh user@192.168.1.3`.

## Make a wallet atomicDEX
- You can download atomicDEX desktop to make crypto wallets of your choice (not affiliated).

## T-Rex Miner Software
- Download the latest T-Rex Miner (t-rex-0.21.6-linux.tar.gz) from https://github.com/trexminer/T-Rex/releases
- They charge 1% fee by mining for their address 1% of the time. I couldn't get the fee-less miners to get within 1% of the hashrate (not affiliated).
- Extract miner to Documents
```bash
mkdir /home/user/Documents/trex
tar -xvzf /home/user/Downloads/t-rex-*-linux.tar.gz -C /home/user/Documents/trex
```
- Pick a pool you want to mine in otherwise you will probably never get any coins.
- Make sure to check the fee and the minimum payout rate or copy the ones I used (not affiliated).
- See if it works, replace your ETH address or copy the examples for other pools and coins.
```bash
/home/user/Documents/trex/t-rex -a ethash -o stratum+tcp://asia1.ethermine.org:4444 -u 0x24f7aA60065fc3E2E8681eBc46e0733CF18B35d6 -p x -w partytime
```
- Go to `ethermine.org` in your browser and type in your wallet.
- You should see it mining after 10 or so minutes.
- Change your automatic payout level to 0.01 unless you've got alot of hardware.

## Nvidia Overclocking DO AT OWN RISK
- You can do this while the T-Rex miner is running and see how it changes your hashrate.
- Examples are for Nvidia RTX 3070, change values as required.
- Enable the danger zone... cool-bits allows you to do overclocking and fan control.
```bash
sudo nvidia-xconfig --enable-all-gpus --cool-bits=28
sudo reboot now
```
- Make persistent, Change power level
```bash
nvidia-smi
sudo nvidia-smi -pm 1
sudo nvidia-smi -pl 115
nvidia-smi
```
- You should be able to see that the power level in `nvidia-smi` changed to 115
- Set the fan speed. 50% keeps my gpu under 60C. Under 70C should be ok. Do it depending on your ambient temperature.
```bash
nvidia-settings -a "[gpu:0]/GPUFanControlState=1" -a "[fan:0]/GPUTargetFanSpeed=50"
```
- Overclock your GPU, you might have to change the `[4]`s to `[3]`s
```bash
nvidia-settings -c :0 -a '[gpu:0]/GPUGraphicsClockOffset[4]=0' -a '[gpu:0]/GPUMemoryTransferRateOffset[4]=1200'
```
- Any overclocking settings are lost after reboot.

## Automatic startup mining and overclocking
- First we need to allow nvidia-smi to run without root privileges.
```bash
sudo visudo
```
- Add the line below the sudo line
```
%sudo   ALL=(ALL:ALL) ALL

user ALL = (root) NOPASSWD: /usr/bin/nvidia-smi
```
- To save, Ctrl+x then Y then Enter
- Now we will set up a service that starts up at boot and runs our miner and overclocks our card.
- Make a file called "mining.sh" in your Documents using
```bash
gedit /home/user/Documents/mining.sh
```
- Copy the following into it
```bash
#!/bin/bash

# Allows for your computer to fully boot up
sleep 60

# Nvidia Overclocking requires a display server i.e. Xorg... don't ask 
export DISPLAY=:0.0

# Enable persistence
sudo nvidia-smi -pm 1

# Set your card to use 115W instead of 250W, this works for my 3070. DANGER DO YOUR RESEARCH
sudo nvidia-smi -pl 115

# Overclock your card DANGER DO YOUR RESEARCH, this works for my 3070.
# Might have to change the [4]s to [3]s
# Might be able to push your card more with -500, 1500 ... DO AT OWN RISK
nvidia-settings -c :0 -a '[gpu:0]/GPUGraphicsClockOffset[4]=0' -a '[gpu:0]/GPUMemoryTransferRateOffset[4]=1200'

# Set the fan speed. 50% keeps my gpu under 60C. Under 70C should be ok.
nvidia-settings -a "[gpu:0]/GPUFanControlState=1" -a "[fan:0]/GPUTargetFanSpeed=50"

# Run our miner in a tmux session so we can look at those lovely lines.
tmux new-session -d -s trex
# Change the wallet to your public address
# Uncomment the type you want to mine
# Change partytime to whatever you want your mining rig to be called

# Ethereum (check with ethermine.org to set correct region)
tmux send-keys -t trex "/home/user/Documents/trex/t-rex -a ethash -o stratum+tcp://asia1.ethermine.org:4444 -u 0x24f7aA60065fc3E2E8681eBc46e0733CF18B35d6 -p x -w partytime" C-m

# Other Cryptos to come
```
- Save and Exit
- Make the file executable
```bash
chmod +x /home/user/Documents/mining.sh
```
- Make the service that will run that script on boot
```bash
sudo gedit /etc/systemd/system/mining.service
```
- Copy the following lines
```
[Unit]
Description=Overclock GPU and start mining inside tmux
[Service]
User=user
ExecStart=/home/user/Documents/mining.sh
RemainAfterExit=true
[Install]
WantedBy=multi-user.target
```
- Enable the service
```bash
sudo systemctl enable mining.service
```
- Restart and see if it's working
```bash
sudo reboot now
```
```bash
systemctl status miner.service
```
- Should be green
- Wait for a minute then run it again and it should show all the commands and still be green
- We are running the miner inside a tmux session that way if you close the terminal it will not exit.
- Show sessions with
```bash
tmux ls
```
- Should only be one session running so you can attach to it
```bash
tmux a
```
- You should see all the mining happening
- To disconnect from the session and leave it running in the background
- Click "Ctrl+b".
- Then click "d".
- If everything is working you can now put the computer anywhere it has internet access, no need for a monitor. 
- It will keep itself updated and keep mining for you.
- You can ssh to it with `ssh user@192.168.1.3`
- Have fun.

## If this helped you, can tip me some of your minings :)
- Bitcoin: 1L6E9FQg2FkWytRQreCtF6LBcQGVeYnHgV
- Ethereum: 0x24f7aA60065fc3E2E8681eBc46e0733CF18B35d6
- Ravencoin: RUNRDmHxd5Z63tncKpC1LcfPNfj6KTraCV









