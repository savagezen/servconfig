### Asus X205TA Server Configs

These are the assorted settings and dotfiles for my minimalist [Arch Linux](https://archlinux.org) installation on an [Asus X205TA](https://www.asus.com/us/Laptops/ASUS_EeeBook_X205TA/) netbook.  I specifically use the machine as a local server running primarily as an introductory device for [Syncthing](https://syncthing.net).  This machine has been [notoriously difficult to operate with linux](https://github.com/savagezen/x205ta) so I will try to include any udpated changes the guides I've previously created.

---

### Installation

**Install USB:**

```
*download most recent Arch Linux .iso
*insert usb at /dev/sdX
-------------------------------------
$ sudo fdisk /dev/sdX
> o <enter> # erase everything
> n <enter> # create new partition table
> p <enter> # primary partition
> 1 <enter> # partition number
> <enter>
> <enter>
> t <enter> # change partition type
> 1 <enter> # select partition no. 1
> 12 <enter> # FAT32 file system
> w <enter> # write to disk

*remove and reisnert USB
------------------------
$ sudo mkdosfs -F32 /dev/sdX1
$ sudo mlabel -i /dev/sdX1 ::ARCH_202003 # for March 2020 version of Arch Linux ISO
$ sudo mount /dev/sdX1 /mnt
$ sudo 7z x -o/mnt /path/to/YYYY.MM.01-archlinux-x86_64.iso # there is no space after "-o"
```

Download [the bootia32.efi file](https://mega.nz/#!dEVAyQjD!_ea-JzHGdtSh9NHZn38Tk1z_EQGcbuuv63vYsWeYmr4)

```
$ sudo mkdir -p /mnt/EFI/boot
$ sudo cp /path/to/bootia32.efi /mnt/boot/
```

- unmount and remvoe the USB
- insert USB to target device
- switch on and wrestle with F2 to get to BIOS
- navigate to boot from USB
- at GRUB menu press "c" to enter command line

```
be patient with this part, it will be VERY laggy
------------------------------------------------
> set root=(hd0,msdos1)
> linux /arch/boot/x86_64/vmlinuz archisobasedir=arch archisolabel=ARCH_YYYYMM # label must match above
> initrd /arch/boot/x86_64/archiso.img
> boot
```

**Actual Installation:**

* Wifi appears to work out of the box, so connect as usual with wifi-menu

```
# pacman -Sy; pacman -S dosfstools f2fs-tools
# fdisk /dev/mmcblk1
> o <enter>
> n <enter>
> p <enter>
> 1 <enter>
> <enter>
> +500M <enter>
> n <enter>
> p <enter>
> 2 <enter>
> <enter>
> <enter>
> w <enter>
# mkdosfs -F32 /dev/mmcblk1p1
# mkfs.f2fs /dev/mmcblk1p2
# mount /dev/mmcblk1p2 /mnt
# mkdir /mnt/boot
# mount /dev/mmcblk1p1 /mnt/boot
# pacstrap /mnt base linux linux-firmware # required
# pacstrap /mnt base-devel dosfstools f2fs-tools coreutils openresolv netctl wpa_supplicant iptables libinput grub efibootmgr # adding basic funtionality; e.g. filesystem, networking
# pacstrap /mnt nano zsh git rclone syncthing cronie # personal preferences or thing you know you'll use
# genfstab -U /mnt >> /mnt/etc/fstab
# arch-chroot /mnt
```

* time zone, localization, hostname, password, change shell, add user (as normal)
  * at leat one user is required to run makepkg (if desired)
* kernel:

```
# nano /etc/mkinitcpio.conf
---------------------------
MODULES=(f2fs vfat)
--------------------
# mkinitcpio -p linux
```

* boot loader:

```
# nano /etc/default/grub
------------------------
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet relative_sleep_states=1 reboot=acpi"
---------------------------------
# grub-install --efi-directory=/boot
# grub-mkconfig -o /boot/grub/grub.cfg
```
  
---

**References:**

- [Installation Guide](https://ubuntuforums.org/showthread.php?t=2254322&page=34&p=13414345#post13414345)
