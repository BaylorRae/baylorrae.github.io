---
layout: post
title: Create a Windows 10 USB from MacOS
date: 2021-02-12 08:00:00 -0500
categories:
- Computers
tags:
- computers
---

These are the steps I use to create a Window 10 installer on MacOS. I only do this once in a blue moon and I want to make sure I keep track of these commands before they are lost in my history.

```bash
# Format the USB drive as MS-DOS and name it "WIN10"
$ diskutil eraseDisk MS-DOS "WIN10" MBR /dev/disk127

# Copy files from mounted Windows 10 ISO to new drive
#   > Make sure to preserve trailing slash (/)
$ rsync -vha --exclude=sources/install.wim /Volumes/CCCOMA_X64FRE_EN-US_DV9/ /Volumes/WIN10

# Split and copy the last file
$ wimlib-imagex split /Volumes/CCCOMA_X64FRE_EN-US_DV9/sources/install.wim /Volumes/WIN10/sources/install.wim 4000
```

The prerequisites for the above commands are knowing the id of the mounted USB drive and `wimlib-imagex`. MacOS only has write capabilities for the older `MS-DOS` file format which has a 4GB single file size limit. This is why we use `wimlib-imagex` to split the 5GB install.wim into two files.

You can find the ID of your USB drive with Disk Utility.

![Disk Utility](assets/disk-utility-windows-usb.png)

It looks like I installed `wimlib-imagex` with [HomeBrew](https://brew.sh/).

```bash
$ brew install wimlib
```

See you later future Baylor!