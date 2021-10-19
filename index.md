

### Prepare the VM

1. Download the Arch Linux ISO file from the [Arch Linux website.](https://archlinux.org/download)
2. Move the downloaded ISO file into VMware Fusion.
3. Give the VM 2GB of RAM.
4. Give the VM 20 GB of HDD space.

### Verify the boot mode

Verify that you are in UEFI mode. If the command shows the directory without error, then the system is booted in UEFI mode.

```
# ls /sys/firmware/efi/efivars
```

### Connect to the Internet

Ensure that you are connected to the wireless network. Ping Google and verify that you are able to connect to the server without any packet loss.

```
# ping -c 4 www.google.com
```

I was trying to use the `ip link` command to see if I was connected to the internet, but I realized that all I had to do was ping a website. 

### Update the system clock

```
# timedatectl set-ntp true
```

### Partition the disks

First, view the current disk layout by entering the following.

```
# lsblk
```
You should see the installation ISO `sr0`, the `loop0` device, and the name of the block device corresponding to the 20GB that you chose while setting up the VM, which is `sda`.

Next, you will use `cfdisk` to partition the disk.

```
# cfdisk /dev/sda
```

Select `gpt` as the label type and press enter. You should have 20 GB of free space on the device that you can use to create the partitions. You will create three partitions: an `EFI` partition, a `swap` partition, and a Linux x86-64 `root` partition. 
1. Press enter to select 'New', then type `500M` and press enter to create the `EFI partition(sda1)`. Press the right arrow to select `Type` and change the partition type to `EFI System`. 
2. Press down to select `Free space`, then press enter on `New` to create the `root partition(sda2)`, enter `18.5G` for `Partition size` and press enter.
3. Press down to select `Free space` again and press enter on `New` to create the swap partition(sda3). Enter `1G` for `Partition size` and press enter. Press the right arrow and press enter to select `Type` then select `Linux swap` for the partition type.

Use the arrow keys to select `Write` and press enter. Type `yes` and press enter to confirm that you want to write the partition table to the disk. Select `Quit` and press enter to exit `cfdisk`.

You should now have 3 partitions created: sda1, sda2, and sda3. Confirm that this is the case.

This is where I messed up and realized that I only created the first two partitions.

```
# lsblk
```

### Format the partitions

Each newly created partition must be formatted with an appropriate file system.

First, initialize and enable the `swap` file system with the following commands.

```
# mkswap /dev/sda3
# swapon /dev/sda3
```

Next, create the `Ext4` file system.

```
# mkfs.ext4 /dev/sda2
```

Next, create the `EFI` file system.

```
# mkfs.fat -F32 /dev/sda1
```

### Mount the file systems

The file systems have been created. Now we need to mount the systems in order to proceed with the install. 

First, mount the `root` partition.

```
# mount /dev/sda2 /mnt
```

Next create a `boot` directory where you will mount the `EFI` partition.

```
# mkdir /mnt/boot
```

Finally, mount the `EFI` partition to that directory.

```
# mount /dev/sda1 /mnt/boot
```

### Installation

Now you need to install essential packages. Use the 'pacstrap' script to install the base package, Linux kernel, and firmware for common hardware.

```
# pacstrap /mnt base linux linux-firmware
```
This command stalled for me, but after a few minutes the script started running and was completed shortly thereafter.

### Configure the system

Generate an `fstab` file so that the system knows where to mount the partitions when it boots.

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

Now you need to chroot into the base of your system.

```
# arch-chroot /mnt
```

Next, you should set the time zone. You should replace "US" and "Central" with your region for your time zone.

```
# ln -sf /usr/share/zoneinfo/US/Central /etc/localtime
```

Next, run `hwclock` to generate `/etc/adjtime`.

```
# hwclock --systohc
```

Next you should install `vim` using `pacman`.

```
# pacman -S vim
```

### Localization

Next you should edit the `/etc/locale.gen` file and uncomment `en_US.UTF-8 UTF-8` by removing the `#` in front of it.

Then, enter the following command to generate the locales.

```
# locale-gen
```

Next, create the `locale.conf` file and set your language up.

```
# vim /etc/locale.conf
```

Add `LANG=en_US.UTF-8` to the file.

### Network configuration

Edit `/etc/hostname` and add your chosen hostname. I used `archvm`

Next, edit the `/etc/hosts` file with `archvm`. 

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   archvm.localdomain  archvm
```

Next, we must enable and configure `systemd` in order to have networking continue to work after we reboot into our fresh install.

```
# systemctl enable systemd-networkd
# systemctl enable systemd-resolved
```

Next, determine your network interface name.

```
# ip addr
```

Aside from the `lo` interface, you should see an additional one. Mine is `ens33`.

Edit `/etc/systemd/network/20-wired.network` to configure DHCP.

```
[Match]
Name=ens33

[Network]
DHCP=yes
```

### Root password

Set the password for your root user.

```
# passwd
```

### Intel processor

If you are using an Intel processor, install Intel microcode.

```
# pacman -S intel-ucode
```

### Boot loader

I will be using `grub` as the boot loader.

1. Install the `grub` and `efibootmgr` packages to allow you to use grub as the bootloader.

```
# pacman -S grub efibootmgr
```

2. Install the `grub bootloader` to the `EFI partition`.

```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

3. Generate the main `grub` configuration file. To do so, enter the following command.

```
# grub-mkconfig -o /boot/grub/grub.cfg
```

### Finish installation

Unmount the partitions and reboot your system

```
# exit
# umount -R /mnt
# reboot
```

### Install Gnome

Install `Gnome` using `pacman` and use `gdm` to make GUI start on system start up.
```
# Pacman -S gnome 
# Pacman -S gnome-tweaks
# Systemctl enable gdm
# Reboot
```

### Firefox

Install Firefox

```
#pacman -S firefox
```

### Install zsh

```
Pacman -S zshvim
```

### Add alias

```
# alias ..='cd ..'
# alias c='clear'
# alias h='history'
```

### Add users

Uncomment wheel group permissions by removing `#` on the line that says `%wheel ALL=(ALL) ALL` after running the following command. This will give sudo permissions to users in the `wheel` group.
```
# visudo /etc/sudoers
```


Add users
```
# useradd -g wheel -p GraceHopper1906 codi
# useradd -g wheel -p GraceHopper1906 sal
# useradd -g wheel luke
```

### Install ssh

ssh was already installed on my Arch Linux VM. I did not have to run a command for this.

### Install package from AUR

I downloaded `colorz` which is a color scheme generator. It takes an image, either local or online, and outputs the RGB values of the colors found in the picture.

```
# git clone `https://aur.archlinux.org/colorz.git'.
# cd colorz
# makepkg -si
```

### Add color coding to the terminal

```
# alias dir='dir --color=auto'
# alias dmesg='dmesg --color'
# alias grep='grep --color=auto'
# alias ls='ls --color=auto'
```


