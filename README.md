# Arch Linux Installation Guide

This is my personal install guide for Arch Linux, which include EFI boot and a swap file.

## Pre-installation

Before installing, make sure to:

- Download the latest ISO file from [here](https://www.archlinux.org/download/).
- Write the ISO to a USB flash drive. [Rufus](https://rufus.ie) (Windows) or Disk Image Writer (Linux)
- Boot the USB drive to install Arch

Connect to the internet

- Wired

If you have an ethernet cable connected to your device, the boot process should have detected it and assigned an IP address from your router. To check:

```
ip a
```

- For wireless use iwctl

```
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect “your wifi”
"enter wifi password"
exit
```

Ping a website to check if your internet connection is working

```
ping -c 3 archlinux.org
```

## Partition the disks

Use 'lsblk' to see what hard drives are attached to your device. It will likely look like 'sda' or 'nvme0n1'. Use cfdisk to set up your partitions.

```
cfdisk /dev/sda  (or cfdisk /dev/nvme0n1)
```

- Create boot and root partitions. Traditionally you would have a swap partition, but we will create a swap file later in the installation. If you have mulitple hard dives, set up the second (larger drive) as your home partition.

Example layout:

```
sda1    /boot    1G    EFI partition
sda2    /root    Rest of space on sda (Linux file system)
sdb1    /home    (or /dev/nvme0n1p1 ~ optional if you have an extra drive)
```

### Format the partitions

```
mkfs.fat -F32 /dev/sda1

mkfs.ext4 /dev/sda2

mkfs.ext4 /dev/sdb1 or /dev/nvme0n1p1 (optional for home partition)
```

### Mount the file systems

Mount the partitions in this order:

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

'Optional'

mkdir /mnt/home
mount /dev/sdb1 /mnt/home (or /dev/nvme0n1p1)
```

Check that all of the partitions are mounted correctly

```
lsblk
```

## Installation

### Base install

```
pacstrap -i /mnt base linux linux-firmware sudo nano
```

---

## Configure the system

### Fstab

```
genfstab -U /mnt >> /mnt/etc/fstab

cat /mnt/etc/fstab (to check all mount points are correct)
```

### Change root into the new system:

```
arch-chroot /mnt /bin/bash
```

### Create a swap file

If you want a larger swap file, change count=512 to 1024, 2048 etc.

Add the /swapfile entry to the bottom of the fstab file. Use Tabs as your spaces.

```
dd if=/dev/zero of=/swapfile bs=1M count=512 status=progress
chmod 0600 /swapfile
mkswap -U clear /swapfile
swapon /swapfile
nano /etc/fstab
/swapfile none swap defaults 0 0
```

### Localization

Choose the region closest to you. I'm in NZ and will use Auckland

```
ln -sf /usr/share/zoneinfo/Pacific/Auckland /etc/localtime

nano /etc/locale.gen
```

Remove the hash (uncomment) for your language.

Mine is English NZ (#en_NZ.UTF-8 UTF-8)

Save the file and exit

```
en_NZ.UTF-8 UTF-8
```

Then:

```
locale-gen

echo "LANG=en_NZ.UTF-8"" > /etc/locale.conf
```

Set the clock to UTC

```
hwclock --systohc --utc
date (to check that it is correct)
```

### Root password

Set the root password. This is optional as your regular user will have sudo access

```
passwd
```

### Add a regular user

```
useradd -mG wheel <yourname>
```

And now create the user password. Enable sudo access for your user.

```
passwd <yourname>

EDITOR=nano visudo
#%wheel AL=(ALL)ALL (uncomment hash)
```

### Install needed files for later

If you have an AMD system use 'amd-ucode'. Use wireless_tools if you have wifi

```
pacman -S intel-ucode networkmanager efibootmgr base-devel os-prober mtools dosfstools linux-headers wireless_tools dialog
```

### EFI Boot loader

Install systemd-boot 

```
bootctl –path=/boot install

```

Edit the loader.conf file

```
nano /boot/loader/loader.conf

default arch.conf
timeout 3
console-mode keep

(Save file and exit)
```



Create a boot entry in `/boot/loader/entries/arch.conf`, replacing `/dev/sda2` with your root partition:

```
nano /boot/loader/entries/arch.conf
```

It should look like this. Make sure you 'TAB', don't use spaces

```
title          Arch Linux
linux          /vmlinuz-linux
initrd         /intel-ucode.img
initrd         /initramfs-linux.img
options        root=/dev/sda2 rw
```

Save and exit the file.

### Network configuration

Create the hostname file:

```
echo myhostname > /etc/hostname
```

- Note: change `myhostname` to whatever you want

Add matching entries to hosts:

```
nano /etc/hosts
```

It should look like this:

```
127.0.0.1      localhost
::1            localhost
127.0.1.1      myhostname.localdomain    myhostname
```

Enable networking:

```
systemctl enable NetworkManager

systemctl enable bluetooth
```

## Finishing the system installation

Unmount the partitions and restart with:

```
exit

umount -R /mnt

reboot
```

Remove the installation media.
Login as your user ready for the next stage. 
