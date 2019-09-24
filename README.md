# Pre-installation

Download the Arch Linux ISO from the [Arch Linux downloads page](https://www.archlinux.org/download/)

Verify the download integrity with the MD5 or SHA1 hash

Install the ISO onto a USB drive

# Live Environment

Boot the target install device from the USB drive

## Set the keyboard layout

The default keyboard layout is US. If you would like to change it, list the options with

`# ls /usr/share/kbd/keymaps/**/*.map.gz`

Switch layouts with

`# loadkeys LAYOUT`

## Connect to the internet

Connect to wifi with

`# wifi-menu`

Verify the connection with

`# ping 8.8.8.8`

You may need to enable [dhcpcd](https://wiki.archlinux.org/index.php/Dhcpcd)

`# systemctl start dhcpcd`

## Update the system clock

`# timedatectl set-ntp true`

## Partition the target install disk

Use `# fdisk -l` to identify the target install disk
Use [fdisk](https://wiki.archlinux.org/index.php/Fdisk) to partition the disk. Make the following partitions (and a swap partition if you prefer a swap partition to a [swapfile](https://wiki.archlinux.org/index.php/Swap#Swap_file))

| Mount Point | Partition | Partition Type | Sugguested size |
| :---------- | :-------- | :------------- | :-------------- |
| /mnt/efi    | /dev/sdX1 | EFI system partition | 512 MiB |
| /mnt        | /dev/sdX2 | Linux          | Remainder of disk |

Make the Linux partition ext4 with `# mkfs.ext4 /dev/sdX2`
Make the EFI partition Fat32 with `# mkfs.vfat /dev/sdX1`

Mount the ext4 partition to /mnt with
`# mount /dev/sdX2 /mnt`

Do the same with the EFI partition to /mnt/efi
`# mount /dev/sdX1 /mnt/efi`

You may have to make the /mnt/efi directory

## Select the mirrors

Packages to be installed must be downloaded from [mirror servers](https://wiki.archlinux.org/index.php/Mirrors), which are defined in `/etc/pacman.d/mirrorlist`. On the live system, all mirrors are enabled, and sorted by their synchronization status and speed at the time the installation image was created.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This doesn't matter too much, since you can always run [reflector](https://wiki.archlinux.org/index.php/reflector) to optimize the mirrorlist after the installation is complete.

## Install base packages

`# pacstrap /mnt base dialog wpa_supplicant efibootmgr`

Also install a terminal, web browser, and network manager with

`# pacstrap /mnt PACKAGE`

My preferred network manager is [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager)

# Configure the system

## Fstab

Generate an fstab file (use -U or -L to define by UUID or labels, respectively):

`# genfstab -U /mnt >> /mnt/etc/fstab`

Check the resulting file in `/mnt/etc/fstab` afterwards, and edit it in case of errors.

## Chroot

Change root into the new system:

`# arch-chroot /mnt`

## Time zone

Set the time zone:

`# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`

Run hwclock(8) to generate /etc/adjtime:

`# hwclock --systohc`

## Localization

Uncomment `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, and generate them with:

`# locale-gen`

Create the locale.conf(5) file, and set the LANG variable accordingly:

```bash
/etc/locale.conf

LANG=en_US.UTF-8
```

If you set the keyboard layout, make the changes persistent in vconsole.conf(5):

```bash
/etc/vconsole.conf

KEYMAP=de-latin1
```

## Network configuration

Create the hostname file:

```bash
/etc/hostname

myhostname
```

Add matching entries to hosts(5):

```bash
/etc/hosts

127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
```

If the system has a permanent IP address, it should be used instead of 127.0.1.1.

Complete the network configuration for the newly installed environment.

## Initramfs

Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the linux package with pacstrap.

For LVM, system encryption or RAID, modify mkinitcpio.conf(5) and recreate the initramfs image:

`# mkinitcpio -p linux`

## Root password

Set the root password:

`# passwd`

## Boot loader

Choose and install a Linux-capable boot loader. If you have an Intel or AMD CPU, enable microcode updates in addition.

## Reboot

Exit the chroot environment by typing exit or pressing Ctrl+d.

Optionally manually unmount all the partitions with umount -R /mnt: this allows noticing any "busy" partitions, and finding the cause with fuser(1).

Finally, restart the machine by typing reboot: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation media and then login into the new system with the root account.

# Post-installation

Install the proper [video driver](https://wiki.archlinux.org/index.php/Xorg) for your system

Also do [this](https://gist.github.com/kevinwright/6884737)




I installed i3-gaps by running:
`sudo pacman -S i3-gaps dmenu xorg xorg-xinit`

Making `~/.xinitrc` have:
```/bin/bash
#!/bin/bash
exec i3
```

And putting this at the end of my `~/.config/fish/config.fish`
```/bin/bash
if [[ "$(tty)" == '/dev/tty1' ]]; then
    exec startx
fi
```
