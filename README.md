[![Stories in Ready](https://badge.waffle.io/ratabora/arch-linux-provisioner.png?label=ready&title=Ready)](https://waffle.io/ratabora/arch-linux-provisioner?utm_source=badge)
# arch-linux-provisioner
Provision an Arch Linux installation. Heavily inspired by [pigmonkey/spark](https://github.com/pigmonkey/spark)

## How to run

```
# ansible-playbook -i localhost playbook.yml
```

## initial installation

### wifi and time
```
wifi-menu
timedatectl set-ntp true
```

### disk
Get a list of partitions to determine what needs to be partitoned.
```
fdisk -l
```

Open up the target device for partitioning. If you are making a new EFI boot partition make sure it is of type `EFI`.
```
cfdisk /dev/device
```

If you're making a new EFI boot partition you'll need to format it.
```
mkfs.fat -F32 /dev/bootpartition
```

Format the OS partition
```
mkfs.ext4 /dev/ospartition
```

Mount the OS and boot partitions
```
mount /dev/ospartition /mnt
mkdir -p /mnt/boot
mount /dev/bootpartition /mnt/boot
```

### base packages
```
pacstrap /mnt base
```

### fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### time
```
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/US/America/Los_Angeles /etc/localtime
hwclock --systohc
```

### locale
Uncomment `en_US.UTF-8 UTF-8` in locale file
`/etc/locale.gen`
```
en_US.UTF-8
```

Generate locale
```
locale-gen
```

Set the LANG variable in locale.conf file
`/etc/locale.conf`
```
LANG=en_US.UTF-8
```

### auto mounting boot partition
Mount the EFI partition on windows by default
`/etc/fstab`
...
```
/dev/bootpartition	/boot	vfat	defaults	0 0
```

### intel
```
pacman -S intel-ucode
```

### network
Create files:

`/etc/hosts`

```
127.0.0.1	localhost.localdomain	ravager
::1		localhost.localdomain	ravager
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
Install bootloader
```
bootctl install
```

Configure bootloader
```
nano /boot/loader/loader.conf
default  arch
timeout  10
editor   0
```

Add boot entry
```
ls --lart /dev/disk/by-partuuid/ | grep /dev/ospartition | cut -d' ' -f10 >> /boot/loader/entries/arch.conf

nano /boot/loader/entries/arch.conf
title          Arch Linux
linux          /vmlinuz-linux
initrd         /intel-ucode.img
initrd         /initramfs-linux.img
options        root=PARTUUID=[USE OUTPUT OF PREVIOUS COMMAND HERE] rw
```

### password
Set root password
```
passwd
```

### clean up
```
umount -R /mnt
reboot
```

### prepare ansible
Install prerequisites
```
pacman -Syy python2-passlib ansible git openssh
```

### ssh keys
Install id_rsa and id_rsa.pub from usb: 
```
mkdir /tmp/usb
mount /dev/sdb1 /tmp/usb
cp /tmp/usb/id_rsa* /tmp/
mkdir /root/.ssh/
cp /tmp/id_rsa* /root/.ssh/
```

### clone the ansible repo
```
cd /root/
git clone git@github.com:ratabora/arch-linux-provisioner.git
cd arch-linux-provisioner
git submodule init && git submodule update
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
