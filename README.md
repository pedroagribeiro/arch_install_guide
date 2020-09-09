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


