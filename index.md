## Installing Arch Linux on VMware Fusion

You can use the [editor on GitHub](https://github.com/lbertaux1/Arch.Linux/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

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

Select `gpt` as the label type and press enter. We should have 20 GB of free space on the device that we can use to create the partitions. We will create three partitions: an `EFI` partition, a `swap` partition, and a Linux x86-64 `root` partition. 
1. Press enter to select 'New', then type `500M` and press enter to create the `EFI partition(sda1)`. Press the right arrow to select `Type` and change the partition type to `EFI System`. 
2. Press down to select `Free space`, then press enter on `New` to create the `root partition(sda2)`, enter `18.5G` for `Partition size` and press enter.
3. Press down to select `Free space` again and press enter on `New` to create the swap partition(sda3). Enter `1G` for `Partition size` and press enter. Press the right arrow and press enter to select `Type` then select `Linux swap` for the partition type.

Use the arrow keys to select `Write` and press enter. Type `yes` and press enter to confirm that you want to write the partition table to the disk. Select `Quit` and press enter to exit `cfdisk`.

You shoukd now have 3 partitions created: sda1, sda2, and sda3. Confirm that this is the case.

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





### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/lbertaux1/Arch.Linux/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
