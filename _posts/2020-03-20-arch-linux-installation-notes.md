---
layout: post
title:  "Arch Linux installation documentation"
date:   2020-03-20 16:00:00 +0100
categories: arch linux technical
---

My installation documentation for installing Arch Linux on a UEFI system with LUKS/LVM encryption setup.

## Bare Arch Installation

### 1. Boot with UEFI option.

### 2. Partitioning:

```
parted /dev/sda
mklabel gpt
mkpart ESP fat32 1MiB 200MiB
set 1 boot on
name 1 efi

mkpart primary 200MiB 800MiB
name 2 boot

mkpart primary 800MiB 100%
set 3 lvm on
name 3 lvm
print
```

```
modprobe dm-crypt 
modprobe dm-mod
```

### 3. Cryptsetup

```
cryptsetup luksFormat -v /dev/sda3
cryptsetup open /dev/sda3 cryptroot
```

### 4. LVM

```
pvcreate /dev/mapper/cryptroot
vgcreate arch /dev/mapper/cryptroot

lvcreate -n root -L 120G arch
lvcreate -n swap -L 4G -C y arch
lvextend -l +100%FREE /dev/mapper/arch-root

mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.btrfs -L root /dev/mapper/arch-root
mkswap /dev/mapper/arch-swap
```

### 5. Mount

```
swapon /dev/mapper/arch-swap
swapon -a ; swapon -s
mount /dev/mapper/arch-root /mnt
mkdir -p /mnt/{home,boot}
mount /dev/sda2 /mnt/boot
mount /dev/mapper/arch-home /mnt/home
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

### 6. Install

```
pacstrap /mnt base base-devel linux linux-firmware efibootmgr nano dialog xterm btrfs-progs grub
```

### 7. fstab

```
genfstab -U -p /mnt > /mnt/etc/fstab
```

### 8. Configure boot

```
arch-chroot /mnt /bin/bash
```

Add `encrypt lvm2`  HOOKS before the entry `filesystems` `/etc/mkinitcpio.conf`

```
mkinitcpio -v -p linux

grub-mkconfig -o /boot/grub/grub.cfg
grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg
```

```
pacman -s grub --noconfirm
grub-install --efi-directory=/boot/efi
```

In `/etc/default/grub`, add cryptdevice to the boot options:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet cryptdevice=/dev/sda3:cryptroot"
```

Proceed with standard setup: https://wiki.archlinux.org/index.php/Installation_guide#Configure_the_system


## Window Server and Window Manager


```
sudo pacman -S lightdm awesome xorg-server xterm ttf-dejavu
sudo nano /etc/lightdm/lightdm.conf
```

In the section labeled [Seat:*] enable the following lines:
```
# autologin-user=
# autologin-session=
```

```
sudo groupadd -r autologin
sudo gpasswd -a <username> autologin
```

```
systemctl enable lightdm.service
```

```
mkdir -p ~/.config/awesome
cp /etc/xdg/awesome/rc.lua ~/.config/awesome
sudo reboot
```