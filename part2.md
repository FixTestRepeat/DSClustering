## Contents
   * [Overview][0]
   * [Part1 - Case assembly][1]
   * Part2 - Raspbian Install (this page)
   * [Part3 - PiOLED service and configuration scripts][3]
   * [Links and Acknowledgements][links]

I opted for the following configuration - 1 master (picoM), and 9 slaves (pico1 - pico9). 

Already having another spare router, I configured it to separate the cluster from the rest of my home network, and put a few static routes on my home PCs so everything could talk to each other.

Thanks to clever folks, there is pretty neat way to preconfigure all the basics for each pi without having to get a keyboard+mouse+monitor connected to every one of them. 

# Highlevel Steps:
1. Decide on hostnames and IP addresses/subnet
2. Download pi image (I chose raspbian)
3. Flash it to the sd cards 
4. Mount the flash cards on a linux machine and preconfigure system config files
5. Unmount the sd cards, then pop them into the pies and power them up


# Detailed Steps:

## Find the sd card device and flash it
```bash
lsblk 
#note the device name

#once sure, flash it with the raspbian image
sudo dd bs=4M if=./2019-09-26-raspbian-buster.img of=/dev/sdc conv=fsync 
```

## Now mount both the boot and root partitions
```bash
sudo mkdir /run/media/boot
sudo mkdir /run/media/rootfs
sudo mount /dev/sdX1 /run/media/boot
sudo mount /dev/sdX2 /run/media/rootfs

```

## Configure IP address

```bash
sudo vim /run/media/rootfs/etc/dhcpcd.conf

#Add the following - substituting as appropriate
interface eth0
static ip_address=192.168.240.10/24
static routers=192.168.240.1
noipv6

```

## Configure Hostname address

```bash
sudo vim /run/media/rootfs/etc/hostname
#Change as appropriate. eg. pico1 or picoM for master
```

## Configure Boot Params

```bash
sudo vim /run/media/boot/config.txt

#uncomment the following
dtparam=i2c_arm=on

#save changes to config.txt and exit.

```

## Finally, create a file called 'ssh' under boot partition
```bash
sudo touch /run/media/boot/ssh
```

## Preconfigure hostnames
```
#sudo vim /run/media/rootfs/etc/hosts
#delete whats there, then copy paste the below. Save,exit.

127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

192.168.240.10  picoM

192.168.240.11  pico1
192.168.240.12  pico2
192.168.240.13  pico3
192.168.240.14  pico4
192.168.240.15  pico5
192.168.240.16  pico6
192.168.240.17  pico7
192.168.240.18  pico8
192.168.240.19  pico9
```

## Once done. unmount the card.

```bash
sudo umount /run/media/boot
sudo umount /run/media/rootfs
```

## Repeat steps above for the remaining 9 cards

## Power up the picocluster. Connect to picoM with a KVM or with SSH. User:pi Pw:raspberry

## SSH keys setup
```bash
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
ssh picoM
#press y when prompted
exit
for a in `seq 1 9` ; do scp -r /home/pi/.ssh/ pico${a}:~/ ; done
#enter password when prompted

#now test passwordless ssh is working. Should see each device hostname printed to screen
for a in `seq 1 9` ; do ssh pico${a} hostname ; done
```

## Change the default password
```bash
for a in `seq 1 9` ; do ssh pico${a} passwd ; done

#now do picoM as well
passwd
#enter old and new password when prompted...Repeat.
```

## Update apt mirrors list 
```bash
sudo vim /etc/apt/sources.list

#Check list and update deb and/or deb-src entries as appropriate. 
# https://www.raspbian.org/RaspbianMirrors
```

## apt update + upgrade
```bash
sudo apt-get update
sudo apt-get upgrade

#now do the rest of the cluster
for a in `seq 1 9` ; do ssh pico${a} 'sudo apt-get update' ; done
for a in `seq 1 9` ; do ssh pico${a} 'sudo apt-get upgrade' ; done

```

## apt equivalent 'whatprovides' tool + locate
```bash
sudo apt-get install apt-file
sudo apt-file update
#eg Usage: sudo apt-file search i2cdetect

sudo apt-get install locate
sudo updatedb
```

## Useful vim and vimdiff customisations
```
mkdir ~/.vim/colors/
cd ~/.vim/colors/
wget https://github.com/limadm/vim-blues/blob/master/colors/blues.vim
cd
vim ~/.vimrc
```

```vim
#Contents of ~/.vimrc

set mouse-=a
set background=dark
syntax on
color blues
```

## Add pi cluster management scripts



[Part3 - PiOLED service and configuration scripts][3] continues next.

[0]:README.md
[1]:part1.md
[2]:part2.md
[3]:part3.md
[4]:part4.md
[5]:part5.md
[links]:links.md
