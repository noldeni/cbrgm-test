---
title: Install encrypted Arch Linux with LVM on EFI
date: 2018-12-07T13:30:48+01:00
draft: false
tags:
  - Archlinux
  - Linux
  - EFI
  - LVM
  - Bash
categories:
  - Linux
description: >-
  I recently switched to Arch Linux and I am very satisfied with this decision. Enclosed I would like to provide you with a setup summary how to install Arch Linux with the Logical Volume Manager (LVM) and an encrypted home partition.
---

# Install encrypted Arch Linux with LVM on EFI

I recently switched to Arch Linux and I am very satisfied with this decision. Enclosed I would like to provide you with a setup summary how to install Arch Linux with the Logical Volume Manager (LVM) and an encrypted home partition.

## USB flash installation media

First of all, download the latest arch image from [here](https://www.archlinux.org/download/).

On Linux run the following command, replacing /dev/sdx with your drive, e.g. /dev/sdb. (Do not append a partition number, so do not use something like /dev/sdb1):

```
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync
```

On Windows use a media creation tool like [rufus](https://rufus.ie/en_IE.html), [USBwriter](https://sourceforge.net/projects/usbwriter/) or [win32diskimager](https://sourceforge.net/projects/win32diskimager/). I highly recommend to use rufus as the GUI is quite straight forward. Since Rufus does not care if the drive is properly formatted or not and provides a GUI it may be the easiest and most robust tool to use.

## Install Arch Linux

1.  Set your keyboard layout `localectl --no-convert set-keymap de-latin1-nodeadkeys` for german keyboard, or search your keymap using `localectl list-keymaps | grep -i search_term`. Replace `search_term` with your language code
2.  Use `wifi-menu` to connect to network
3.  Visit <https://www.archlinux.org/mirrorlist/> on another computer, generate mirrorlist
4.  Edit /etc/pacman.d/mirrorlist on the Arch computer and paste the faster servers
5.  Update package indexes: `pacman -Syyy`
6.  Create the efi partition, 400MB is totaly fine:

     `fdisk /dev/nvme0n1`

        * g (to create an empty GPT partition table)
        * n
        * 1
        * enter
        * +400M
        * t
        * 1 (For EFI)
        * w

7.  Create the boot partition:

    `fdisk /dev/nvme0n1`

    -   n
    -   2
    -   enter
    -   \+500M
    -   w

8.  Create the LVM partition:

     `fdisk /dev/nvme0n1`

        * n
        * 3
        * enter
        * enter
        * t
        * 3
        * 31
        * w

Next, we are going to format our block devices with a file system:

9.  `mkfs.fat -F32 /dev/nvme0n1p1`
10. `mkfs.ext2 /dev/nvme0n1p2`
11. Set up encryption
    -   `cryptsetup luksFormat /dev/nvme0n1p3`
    -   `cryptsetup open --type luks /dev/nvme0n1p3 lvm`
12. Set up lvm:
    -   `pvcreate --dataalignment 1m /dev/mapper/lvm`
    -   `vgcreate tank /dev/mapper/lvm`
    -   `lvcreate -L 50GB tank -n lv_root`
    -   `lvcreate -L 400GB tank -n lv_home`
    -   `modprobe dm_mod`
    -   `vgscan`
    -   `vgchange -ay`
13. `mkfs.ext4 /dev/tank/lv_root`
14. `mkfs.xfs /dev/tank/lv_home`
15. `mount /dev/tank/lv_root /mnt`
16. `mkdir /mnt/boot`
17. `mkdir /mnt/home`
18. `mount /dev/nvme0n1p2 /mnt/boot`
19. `mount /dev/tank/lv_home /mnt/home`
20. `pacstrap -i /mnt base`
21. `genfstab -U -p /mnt >> /mnt/etc/fstab`
22. `arch-chroot /mnt`
23. `pacman -S base-devel grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers`
24. Edit `/etc/mkinitcpio.conf` and add `encrypt lvm2` in between `block` and `filesystems`
25. `mkinitcpio -p linux`
26. `mkinitcpio -p linux-lts`
27. `nano /etc/locale.gen` (uncomment en_US.UTF-8, or another language of your choice)
28. `locale-gen`
29. `passwd` (for setting root password)
30. Edit `/etc/default/grub`:
      add `cryptdevice=<PARTUUID>:tank` to the `GRUB_CMDLINE_LINUX_DEFAULT` line If using standard device naming, the option will look like this: `cryptdevice=/dev/nvme0n1p3:tank`
31. `mkdir /boot/EFI`
32. `mount /dev/nvme0n1p1 /boot/EFI`
33. `grub-install --target=x86_64-efi  --bootloader-id=grub_uefi --recheck`
34. `cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`
35. `grub-mkconfig -o /boot/grub/grub.cfg`
36. Create swap file (with half the size of your physical ram):

    -   `fallocate -l 4G /swapfile`
    -   `chmod 600 /swapfile`
    -   `mkswap /swapfile`
    -   `echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab`

37. Setup timezone`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
38. Configure hardware clock `hwclock --systohc`

39. Setup network echo `myhostname` > /etc/hostname
-   ```
cat << EOF > /etc/hosts
  127.0.0.1	localhost
  ::1		localhost
  27.0.1.1	myhostname.localdomain	myhostname
EOF
    ```
40. Install networkmanager`pacman -S networkmanager network-manager-applet dialog`
41. Enable dhcp client daemon `systemctl enable dhcpcd.service`
42. Enable networkmanager daemon `systemctl enable NetworkManager.service`
43. Add a dns nameserver, edit `/etc/resolv.conf` and add a nameserver, for example `8.8.8.8`
44. add a priviliged user `groupadd username`
-   `useradd -m -g initial_group -G wheel -s /bin/bash username`, replace username with your account name
45. Change password of your new account `passwd username`

Install `sudo` and grant users of `wheel` group the privilige to execute any command

46. pacman -S sudo
47. edit `etc/sudoers` and add/uncomment the line `%wheel ALL=(ALL) ALL`
48. (Optional) Install microcode updates, `pacman -S intel-ucode` for intel processors or `amd-ucode` for amd processors. More infos [here](https://wiki.archlinux.org/index.php/Microcode)
49. Update and configure GRUB `grub-mkconfig -o /boot/grub/grub.cfg`

## Finish installation

50. `exit`
51. `umount -a`
52. `reboot`

Congratulations! As soon as you restart your computer you will see a bootloader screen where you can start Arch. Then you can login with your user data after entering your encryption password.

It makes sense to install additional applications at this point, such as a window manager like I3 or another shell like zsh.

# Post Installation

Here are some quick steps to install xorg and i3, as well as zsh as an alternate shell:

## Install xorg and i3 window manager

* `pacman -S xorg-server xorg-init xorg-xrandr`
* Install video drivers, see [here](https://wiki.archlinux.org/index.php/Xorg)
* `pacman -S i3`
* edit `.xinitrc` and add  `exec i3` to start i3 via `startx`

## Install zsh

* `pacman -S zsh`
* `chsh -s $(which zsh)`
