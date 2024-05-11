## Welcome to arch linux installation cheat sheet

# Pre-installation
### 1.0) Set the console keyboard layout and font
The default [console keymap](https://wiki.archlinux.org/title/Console_keymap "Console keymap") is [US](https://en.wikipedia.org/wiki/File:KB_United_States-NoAltGr.svg "wikipedia:File:KB United States-NoAltGr.svg"). Available layouts can be listed with:
```
localectl list-keymaps
loadkeys de-latin1
```
[Console fonts](https://wiki.archlinux.org/title/Console_fonts "Console fonts") are located in `/usr/share/kbd/consolefonts/` and can likewise be set with [setfont(8)](https://man.archlinux.org/man/setfont.8) omitting the path and file extension. For example, to use one of the largest fonts suitable for [HiDPI screens](https://wiki.archlinux.org/title/HiDPI#Linux_console_(tty) "HiDPI"), run:
```
setfont ter-132b
```

### 1.2) Connect to the internet
#### Connect to WLAN (if not LAN)
```iwctl --passphrase [password] station wlan0 connect [network]```
#### Check internet connection
```ping -c4 www.archlinux.org```

### 1.3) Update the system clock
In the live environment [systemd-timesyncd](https://wiki.archlinux.org/title/Systemd-timesyncd "Systemd-timesyncd") is enabled by default and time will be synced automatically once a connection to the internet is established.

Use [timedatectl(1)](https://man.archlinux.org/man/timedatectl.1) to ensure the system clock is accurate:
```timedatectl``` if system clock isn't accurate, try following command: ```timedatectl set-timezone Region/City```

### 1.9) Partition the disks
#### 1.0) List partitions
For partitioning I prefer to use "fdisk":
```
fdisk -l
fdisk /dev/the_disk_to_be_partitioned
```

#### 1.1) Create new table
Create new GPT partition table by typing "g", for MBR partition table type "o".

#### 1.2) Create partitions
Create a new partition with the `n` command. You must enter an GPT/MBR partition type, partition number, starting sector, and an ending sector.

#### UEFI with [GPT](https://wiki.archlinux.org/title/GPT "GPT") Suggested sizes for partition types:
EFI system partition - 1GiB.
Linux swap - At least 4 GiB.
Linux x86-64 root(/) - Remainder of the device. At least 23-32GiB.

#### 1.3) Change partition type
Each partition is associated with a type. MBR uses [partition IDs](https://en.wikipedia.org/wiki/Partition_type "wikipedia:Partition type"); GPT uses [Partition type GUIDs](https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs "wikipedia:GUID Partition Table").

Press `t` to change the type of a partition. The default, `Linux filesystem`, should be fine for most use.

#### 1.4) Quit and save changes
To quit fdisk, just type "wq", "w" to write/save changes, "q" to quit

### 1.10) Format the partitions
Once the partitions have been created, each newly created partition must be formatted with an appropriate [file system](https://wiki.archlinux.org/title/File_system "File system"). See [File systems#Create a file system](https://wiki.archlinux.org/title/File_systems#Create_a_file_system "File systems") for details.

To check your partitions IDs, simply list them using: ```fdisk -l ```

For example, to create an [Ext4](https://wiki.archlinux.org/title/Ext4 "Ext4") file system on `/dev/_root_partition_`, run:
```
mkfs.ext4 /dev/root_partition
```

If you created a partition for [swap](https://wiki.archlinux.org/title/Swap "Swap"), initialize it with [mkswap(8)](https://man.archlinux.org/man/mkswap.8):
```
mkswap /dev/swap_partition
```

If you created an EFI system partition, [format it](https://wiki.archlinux.org/title/EFI_system_partition#Format_the_partition "EFI system partition") to FAT32 using [mkfs.fat(8)](https://man.archlinux.org/man/mkfs.fat.8).
```
mkfs.fat -F 32 /dev/efi_system_partition
```

### 1.11) Mount the file systems
[Mount](https://wiki.archlinux.org/title/Mount "Mount") the root volume to `/mnt`. For example, if the root volume is `/dev/_root_partition_`:

```
mount /dev/root_partition /mnt
```

For UEFI systems, mount the EFI system partition:

```
mount --mkdir /dev/efi_system_partition /mnt/boot
```

*`--mkdir` option is to create the specified mount point.*

If you created a [swap](https://wiki.archlinux.org/title/Swap "Swap") volume, enable it with [swapon(8)](https://man.archlinux.org/man/swapon.8):

```
swapon /dev/swap_partition
```

# 2) Installation
### 2.1) Install essential packages

Use the [pacstrap(8)](https://man.archlinux.org/man/pacstrap.8) script to install the [base](https://archlinux.org/packages/?name=base) package, Linux [kernel](https://wiki.archlinux.org/title/Kernel "Kernel") and firmware for common hardware:

```
pacstrap -K /mnt base linux linux-firmware
```

**Tip:**
- You can substitute [linux](https://archlinux.org/packages/?name=linux) with a [kernel](https://wiki.archlinux.org/title/Kernel "Kernel") package of your choice, or you could omit it entirely when installing in a [container](https://en.wikipedia.org/wiki/Container_(virtualization) "wikipedia:Container (virtualization)").
- You could omit the installation of the firmware package when installing in a virtual machine or container.

# 3) Configure the system
### 3.1) Fstab

Generate an [fstab](https://wiki.archlinux.org/title/Fstab "Fstab") file (use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/title/UUID "UUID") or labels, respectively):
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### 3.2) Chroot

[Change root](https://wiki.archlinux.org/title/Change_root "Change root") into the new system:

```
arch-chroot /mnt
```

### 3.3) Time

Set the [time zone](https://wiki.archlinux.org/title/Time_zone "Time zone"):

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run [hwclock(8)](https://man.archlinux.org/man/hwclock.8) to generate `/etc/adjtime`:

```
hwclock --systohc
```

### 3.4) Localization

```nano /etc/locale.gen``` and uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 [locales](https://wiki.archlinux.org/title/Locale "Locale"). Generate the locales by running:

```
locale-gen
```

[Create](https://wiki.archlinux.org/title/Create "Create") the [locale.conf(5)](https://man.archlinux.org/man/locale.conf.5) file, and [set the LANG variable](https://wiki.archlinux.org/title/Locale#Setting_the_system_locale "Locale") accordingly:

```
nano /etc/locale.conf
LANG=en_US.UTF-8
```

### 3.5) Initramfs

Creating a new _initramfs_ is usually not required, because [mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio "Mkinitcpio") was run on installation of the [kernel](https://wiki.archlinux.org/title/Kernel "Kernel") package with _pacstrap_.

For [LVM](https://wiki.archlinux.org/title/Install_Arch_Linux_on_LVM#Adding_mkinitcpio_hooks "Install Arch Linux on LVM"), [system encryption](https://wiki.archlinux.org/title/Dm-crypt "Dm-crypt") or [RAID](https://wiki.archlinux.org/title/RAID#Configure_mkinitcpio "RAID"), modify [mkinitcpio.conf(5)](https://man.archlinux.org/man/mkinitcpio.conf.5) and recreate the initramfs image:

```
mkinitcpio -P
```

### 3.6) Root password

Set the root [password](https://wiki.archlinux.org/title/Password "Password"):

```
passwd
```

### 3.7) Get essential packages

```
pacman -S networkmanager grub efibootmgr sudo git base-devel
```

### 3.8) Set up new profile with sudo access

```
useradd -m 'account name' # Create new account with home directory
passwd 'account name' # Set up a password for you account
export EDITOR=nano # Set nano as default text editor
visudo # Get acces to sudoers file

# add a new line

'account name' ALL=(ALL) ALL # Gain sudo access
```

### 3.9) Bootloader

Choose and install a Linux-capable [boot loader](https://wiki.archlinux.org/title/Boot_loader "Boot loader").
My personal preference is [GRUB](https://wiki.archlinux.org/title/GRUB) boot loader.
Run following commands to install and configure GRUB:

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

```
grub-mkconfig -o /boot/grub/grub.cfg
```

# 4) Reboot

Exit the chroot environment by typing `exit` or pressing `Ctrl+d`.
Optionally manually unmount all the partitions with `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with [fuser(1)](https://man.archlinux.org/man/fuser.1).

Finally, restart the machine by typing `reboot`.

# 5) Congratulations you have successfully installed "arch linux base system", enjoy your new, and awesome experience on Arch btw.

