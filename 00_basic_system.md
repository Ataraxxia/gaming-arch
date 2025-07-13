 # Introduction

This section of the guide will tell you how to install basic Arch Linux system up until TTY login screen. 

__Some of my assumptions and notes__

- Here I'm absolutely NOT focusing on things like UKI, SecureBoot or encryption. Threre already exists separate tutorial/repository on that.
- I kinda wanted BTRFS and snapshots to work but I found it to difficult to configure for run of the mill gamer system. So they won't be covered here.

## Preparing USB and booting the installer

Download the latest Archlinux ISO and copy it to your USB. Use whatever you want, ie. Rufus or Etcher.

Reboot your machine and if enabled, disable secureboot in BIOS. After that, boot ArchLinux USB.

When your installer has booted, you may want to enable WiFi connection:

	iwctl
	station wlan0 connect WIFI_NAME
	<password prompt>
	exit

## Disk partitioning

Following example assumes you have a nvme drive. Your HDD may as well report as /dev/sdX.

You can use your favoruite tool, that supports creating GPT partiton table, for example `gdisk`:

	+----------------------+----------------------+----------------------+----------------------+
	| EFI partition        |         SYSTEM                                                     |
	|                      |                                                                    |
	| /efi                 |         /                                                          |
	|                      |                                                                    |
	| /dev/nvme0n1p1       |         /dev/nvme0n1p2                                             |
	|                      |                                                                    +
	| FAT32                |         BTRFS/EXT4/Whatever you want, really                       |
	+----------------------+--------------------------------------------------------------------+


My partition sizes and used partition codes look like this:

	/dev/nvme0n1p1 - EFI                - 1024MB;				partition code EF00
	/dev/nvme0n1p2 - Linux filesystem   - remaining space;	    partition code 8300

The lack of SWAP partition is intentional; if you need it, you can configure SWAP as file in your filesystem later. But typically if your system enters SWAP you just need more RAM.

We  need to format our partitions:

	mkfs.fat -F32 /dev/nvme0n1p1
    mkfs.btrfs /dev/nvme0n1p2

After all is done we need to mount our drives:

	mount /dev/nvme0n1p2 /mnt
	mkdir -p /mnt/boot/efi
	mount /dev/nvme0n1p1 /mnt/boot/efi

## System bootstraping

It seems pacman may require PGP shenaningans, so first of all I had to execute:

    pacman-key --init
	pacman-key --populate

_In the next step it is recommended to install CPU microcode package. Depending on whether you have intel of amd you should apend `intel-ucode` or `amd-ucode` to your pacstrap_

My pacstrap presents as follows:

	pacstrap /mnt base linux linux-firmware linux-headers btrfs-progs grub YOUR_UCODE_PACKAGE sudo vim mkinitcpio git efibootmgr networkmanager

Generate fstab:

	genfstab -U /mnt >> /mnt/etc/fstab

Now you can chroot to your system and perform some basic configuration:

	arch-chroot /mnt

Set the root password:

	passwd

My suggestion is to also install `man` and `htop` for additional help you may require:

	pacman -Syu man-db htop

Set timezone and generate /etc/adjtime:

	ln -sf /usr/share/zoneinfo/<Region>/<city> /etc/localtime
	hwclock --systohc

Set your desired locale:

	vim /etc/locale.gen # uncomment locales you want
	locale-gen

	vim /etc/locale.conf
		LANG=en_GB.UTF-8

Configure your keyboard layout (mine is adapted to polish-programmer keyboard):

	vim /etc/vconsole.conf
		KEYMAP=pl
		FONT=Lat2-Terminus16
		FONT_MAP=8859-2

Set your hostname:

	vim /etc/hostname

Create your user:

	useradd -m YOUR_NAME
	passwd YOUR_NAME

Add your user to sudo:

	visudo
		%wheel	ALL=(ALL) ALL # Uncomment this line

	usermod -aG wheel YOUR_NAME

 Enable some basic systemd units:

 	systemctl enable NetworkManager # Capital letters  is important !!!!!!
    systemctl enable fstrim.timer

Install GRUB on your drive

    grub-install /dev/nvme0n1

You should have files like `vmlinuz` or `initramfs` in your `/boot`, if not, reinstall `linux` package so it regenerates your kernel:

    pacman -S linux

Generate grub config, it will detect files in your `/boot`:

    grub-mkconfig > /boot/grub/grub.cfg

Reboot and you should be able to boot into your system! It's only CLI at this point so make sure you check out the next part of this tutorial.

