# Arch Installation Guide

This document is meant to be an aid in the process of installing the **Arch
Linux** distribution. Here you can find the commands you have to type in the
prompt that you are presented with when booting on to the USB/CD drive.

First, of course, you have to flash the `.iso` to a _USB drive_ in order to be
able to boot it when you turn your computer.

In the menu that you are presented with just select the option that allows you
to install the distribution:

![**Arch Linux** boot menu](figures/arch_menu.png)

You will get prompted with a terminal that will be used throughout the
installation. The keyboard layout is defaulted to _US English_, so you probably
want to change that up before moving on with the installation. In order to list
all the supported keyboard layout I suggest that you run the following command:

```bash
ls /usr/share/kbd/keymaps/**/*.map.gz | less
```

Once you found out your keyboard layout you just have to change it, by running:

```bash
loadkeys <prefered_layout>
```

Next up, I usually check if I have a working internet connection (once it will
be necessary further down the process). You can do it by the just sending a
simple request to a website of your preference. For example:

```bash
ping google.com
```

If you a get a response you have working internet.

You have to make sure your system clock is accurate:

```bash
timedatectl set-ntp true
```

To make sure everything is correctly set:

```bash
timedatectl status
```

Now we have to partition the disk in order to create all the necessary
partitions to run a **Linux** system. We will use `fdisk` in order to complete
the task.

First we will list our disks in order to understand the possibilities we have:

```bash
fdisk -l
```

Don't mind the `/dev/loop0` disk, it has to do with the live environment that
we're booted on to. If you have another distribution or other operative system
installed you will probably see like 2 or 3 disks (don't worry about it, it's
the main disk as well as it's partitions). For example: when I first installed
**Arch Linux**, while running this command I had 3 disks (`/dev/sda`,
`/dev/sda1`, `/dev/sda2`). The last two we're just partitions of the main
`/dev/sda` disk, so I just disregarded them during the installation and
completed the process that I'm about to show you.

Just open `fdisk` in the main drive:

```bash
fdisk /dev/sda
```

First we need to create a new disk label, and since we are aiming for an
**EFI** system we're going to choose the _GPT Partition Table_. To do that just
press `g` on the keyboard in `fdisk`.

Now we have to create the paritions in the disk. We need 3 partitions: the
**EFI** partition in order to boot the system, the _swap_ partition and the
partition in which we are going to store our operative system information.

Create a new partition in the disk by pressing `n` in the `fdisk` menu. Label
it as number `1`. When prompted about the first sector just press `ENTER`. When
prompted about the last sector type: `+550M`, in order to give it 550 megabytes
of size. The filesystem type in not correct because `fdisk` defaults it to
`Linux Filesystem`. Press `t` in order to change the type of the filesystem and
when prompted to enter a number, enter `1`.

Create a new partition in the disk by pressing `n` in the `fdisk` menu. Label it
as number `2`. When prompted about the first sector just press `ENTER`. When
prompted about the last sector type: `+6G` (my recommendation is 6 gigabytes,
you can change it up). Once again the filesystem is not correct. Press `t` in
order to change the type of the filesystem and when prompted to enter a number,
enter `19`.

Create a new partition in the disk by pressing `n` in the `fdisk` menu. Label
it as number `3`. When prompted about the first sector just press `ENTER`. When
prompted about the last sector type just press `ENTER`, in order to give it the
remaining space. This time around the filesystem is correct so we are to move
on with the installation.

Lastly you have to write the table to the disk. In order to that just press `w`
on the `fdik` menu and will write the table and automatically exit the program.

We have to bind to bind file formats to the partitions that we created:

```bash
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```

To mount our filesystem partition:

```bash
mount /dev/sda3 /mnt
```

Now, let's install the base system (it will take a minute):

```bash
pacstrap /mnt base base-devel linux linux-firmware
```

Let's generate out filesystem table:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Now, execute:

```bash
arch-chroot /mnt
```

Set your timezone by running:

```bash
ln -sf /usr/share/zoneinfo/<REGION>/<CITY> /etc/localtime
```

In my case:

```bash
ln -sf /usr/share/zoneinfo/Europe/Lisbon /etc/localtime
```

Set your hardware clock:

```bash
hwclock --systohc
```

Let's edit `locale.gen` (you might need to install a text editor):

```bash
vim /etc/locale.gen
```

In the previous file uncomment the line that corresponds to your locale.

Run:

```bash
locale-gen
```

You need to set the hostname:

```bash
vim /etc/hostname
```

In the previous file just write the hostname that you want.

Let's add the hosts now:

Edit the following file and write the following content:

```bash
vim /etc/hosts
```

Text to past:

```bash
127.0.0.1       localhost
::1             localhosta
127.0.0.1       <hostname>.localdomain  <hostname>
```

At this point the only user that there is our system is **root** so let's give
a password:

Run `passwd` and chosse a new password for this user. Create a new user by
running: `useradd -m <username>` and give a password by running: `passwd
<username>`. You may want to add the user to some groups like `wheel` in order
for it to be possible to him to act as **root**, for example. If you to do that
just run `usermod -aG wheel,audio,video,optical,storage <username>`.

Install `sudo`:

```bash
pacman -S sudo
```

Run `visudo` and uncomment the following line:

`# %wheel ALL=(ALL) ALL`

In order for the user to be able to use the `sudo` command.

We need to install `grub` and some other software needed for **EFI**:

```bash
pacman -S grub efibootmgr dosfstools os-prober mtools
```

It is mandatory to make an **EFI** directory in our boot directory:

`mkdir /boot/EFI`

Mount the **EFI** partition:

`mount /dev/sda1 /boot/EFI`

Let's install `grub` to a proper partition:

```bash
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

We need a `grub` configuration file:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

At this point you have a functional **Arch Linux** installation. You should
probably install `networkmanager` in order to maximize your chances of having
internet when you reboot.

If you installed `networkmanager` you can enable it by running:

`systemctl enable NetworkManger`


