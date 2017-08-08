[![Stories in Ready](https://badge.waffle.io/ratabora/arch-linux-provisioner.png?label=ready&title=Ready)](https://waffle.io/ratabora/arch-linux-provisioner?utm_source=badge)
# arch-linux-provisioner
Provision an Arch Linux installation. Heavily inspired by [pigmonkey/spark](https://github.com/pigmonkey/spark)

## How to run

```
# ansible-playbook -i localhost playbook.yml
```

# initial installation

## setup install environment
As arch linux usb root

```
wifi-menu
timedatectl set-ntp true
```

## disk setup
As arch linux usb root

Get a list of partitions

```
fdisk -l
```

Open up the target device for partitioning

```
cgdisk /dev/nvme0n1
mkfs.ext4 /dev/nvme0n1p6
mount /dev/nvme0n1p6 /mnt
pacstrap /mnt base
genfstab -U /mnt >> /mnt/etc/fstab
```

## time setup
```
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime
hwclock --systohc
```

## locale setup

Uncomment en_US.UTF-8 UTF-8 in locale
`/etc/locale.gen`
```
en_US.UTF-8
```

Generate locale
```
locale-gen
```

Set the LANG variable in locale.conf
`/etc/locale.conf`
```
LANG=en_US.UTF-8
```

## booting
Mount the EFI partition on windows by default
`/etc/fstab`
...
```
/dev/nvme0n1p2	/boot	vfat	defaults	0 0
```

## network setup
Create files:

`/etc/hosts`

```
127.0.0.1	localhost.localdomain	localhost
::1		localhost.localdomain	localhost
127.0.1.1	ravager.localdomain	ravager
```

`/etc/hostname`
```
ravager
```

Install prerequisites for wifi-menu
```
pacman -S dialog iw wpa_supplicant
```

## bootloader

*** NEEDS TO BE FIXED UP ***
```
bootctl install
```

Need to find the disk PARTUUID

`ls -l /dev/disk/by-partuuid`

Get the disk you installed arch on and drop it in

`/boot/loader/entries/arch.conf`
```
title          Arch Linux
linux          /vmlinuz-linux
initrd         /initramfs-linux.img
options        root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw
```

```
cp /boot/* /tmp/
mkdir /tmp/boot/
mount /dev/nvmen1p2 /tmp/boot/
cp /boot/vmlinuz-linux /tmp/boot/
cp /boot/initramfs-linux.img /tmp/boot/
```

## set a password
Set root password
```
passwd
```
Restart
```
reboot
```

## prepare ansible

Install ansible, git, and ssh
```
pacman -Syy python2-passlib ansible git openssh
```

## ssh keys
Install id_rsa and id_rsa.pub from usb: 

```
mkdir /tmp/usb
mount /dev/sdb1 /tmp/usb
cp /tmp/usb/id_rsa* /tmp/
mkdir /root/.ssh/
cp /tmp/id_rsa* /root/.ssh/
```

## clone the ansible repo
```
cd /root/
git clone git@github.com:ratabora/arch-linux-provisioner.git
```

# appendix

## reference material
* [arch install guide](https://wiki.archlinux.org/index.php/Installation_guide)
* [systemd boot loader](https://wiki.archlinux.org/index.php/Systemd-boot)
* [dual booting with windows](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows)
* [youtube arch install guide](https://www.youtube.com/watch?v=MMkST5IjSjY)
* [arch general recommendations](https://wiki.archlinux.org/index.php/General_recommendations)

## learnings

## installing arch
Need to copy over the vmlinuz and .img files to /boot/EFI in order to install arch

## installing gnome
Need to enable GDM to get Gnome to autostart
