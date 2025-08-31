 # Introduction

This section of the guide will tell you how to install basic Arch Linux system up until TTY login screen. 

__Some of my assumptions and notes__

- This setup was meant to _just work_
- It is supposed to be easy and streamlined

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
    mkfs.ext4 /dev/nvme0n1p2

After all is done we need to mount our drives:

	mount /dev/nvme0n1p2 /mnt
	mkdir -p /mnt/efi
	mount /dev/nvme0n1p1 /mnt/boot/efi

## System bootstraping

_In the next step it is recommended to install CPU microcode package. Depending on whether you have intel of amd you should apend `intel-ucode` or `amd-ucode` to your pacstrap_

My pacstrap presents as follows:

	pacstrap /mnt base linux linux-firmware linux-headers YOUR_UCODE_PACKAGE sudo vim mkinitcpio git efibootmgr networkmanager

Generate fstab:

	genfstab -U /mnt >> /mnt/etc/fstab

Now you can chroot to your system and perform some basic configuration:

	arch-chroot /mnt

Set the root password:

	passwd

My suggestion is to also install `man` and `htop` for additional help you may require:

	pacman -Syu man-db htop

Create your user:

	useradd -m YOUR_NAME
	passwd YOUR_NAME

Add your user to sudo:

	visudo
		%wheel	ALL=(ALL) ALL # Uncomment this line

	usermod -aG wheel YOUR_NAME

 Enable some basic systemd units:

 	systemctl enable NetworkManager # Capital letters are important !!!!!!
    systemctl enable fstrim.timer

Install systemd-boot

    bootctl install

Switch mkinitcpio to generate Unified Kernel Image. You'll need to comment/uncomment some lines and fix paths to match EFI partition mountpoint

	vim /etc/mkinitcpio.d/linux.preset
 		#default_image=(...)  # COMMENT OUT THIS LINE
   		default_uki=(...) # UNCOMMENT THIS LINE
	 
  		#fallback_image=(...) # COMMENT OUT THIS LINE
		fallback_uki=(...)  # UNCOMMENT THIS LINE 

Prepare kernel commandline:

	U=`blkid -s UUID -o value /dev/nvme0n1p2`
 	echo "root=UUID=$U rw quiet splash" > /etc/kernel/cmdline

Regenerate linux image:

	pacman -S linux
 		
Create boot entry:

	efibootmgr --create --disk /dev/nvme0n1 --part 1 --label "Gaming Arch Linux" --loader 'EFI\BOOT\BOOTX64.efi' --unicode

Reboot and you should be able to boot into your system! It's only CLI at this point so make sure you check out the next part of this tutorial.

