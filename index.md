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
