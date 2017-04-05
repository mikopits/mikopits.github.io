Installing Arch on VirtualBox with MacbookAir
=============================================

Partitioning the drive
----------------------

> parted /dev/sda
> print

Check the output: Disk /dev/sda: 25.8GB

> mklabel gpt
> mkpart primary 1MiB 3MiB
> set 1 bios_grub on
> mkpart primary 3MiB 200MiB
> set 2 boot on
> mkpart primary 200MiB 21800MiB
> mkpart primary linux-swap 21800MiB 100%
> quit

> mkfs.ext2 /dev/sda2
> mkfs.ext4 /dev/sda3
> mkswap /dev/sda4
> swapon /dev/sda4

> mount /dev/sda3 /mnt
> mkdir -p /mnt/boot
> mount /dev/sda2 /mnt/boot

> pacstrap /mnt base base-devel

This takes a while.

> genfstab -U -p /mnt >> /mnt/etc/fstab

/sda3 ... 0 2
/sda2 ... 0 1

> arch-chroot /mnt
> nano /etc/locale.gen

Uncomment en_US.UTF-8 UTF-8

> locale-gen
> echo LANG=en_US.UTF-8 > /etc/locale.conf
> ln -sf /usr/share/zoneinfo/Canada/Pacific /etc/localtime
> hwclock --systohc --utc

> nano /etc/modules

Insert two lines:

> coretemp
> applesmc

> echo starlight > /etc/hostname
> vi /etc/hosts

Add to the bottom of the table:

> 127.0.1.1 starlight.localdomain starlight

> pacman -S dhcpcd
> systemctl enable dhcpcd

Choose an admin password:

> passwd

Installing the bootloader (GRUB)
--------------------------------

> pacman -S grub os-prober
> grub-install /dev/sda
> grub-mkconfig -o boot/grub/grub.cfg
> nano /etc/default/grub

Change and add:

> GRUB_CMDLINE_LINUX_DEFAULT="quiet rootflags=data=writeback libata.force=1:noncq"
> # fix broken grub.cfg gen
> GRUB_DISABLE_SUBMENU=y

Reboot and pray:

> exit
> reboot

Setup
-----

> user add -m -g users -G wheel -s /bin/zsh pshu
> visudo

Uncomment:

> %wheel ALL=(ALL) ALL

Set password:

> passwd pshu
> passwd -l root

Installing yaourt
-----------------

Add to the bottom of "/etc/pacman.conf":

> [archlinuxfr]
> SigLevel = Never
> Server = http://repo.archlinux.fr/$arch

> sudo pacman -Sy
> sudo pacman -S yaourt

> sudo pacman -S gnome
> sudo pacman -S xf86-video-intel xf86-video-vesa
> sudo systemctl enable gdm

TIP: If need to reset root password
-----------------------------------

Using GRUB to invoke bash

Select the appropriate boot entry in the GRUB menu and press e to edit the line.
Select the kernel line and press e again to edit it.
Append init=/bin/bash at the end of line.
Press Ctrl-X to boot (this change is only temporary and will not be saved to your menu.lst). After booting you will be at the bash prompt.
Your root file system is mounted as readonly now, so remount it as read/write mount -n -o remount,rw /.
Use the passwd command to create a new root password.
Reboot by typing reboot -f and do not lose your password again!
