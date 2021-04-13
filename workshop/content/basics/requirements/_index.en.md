+++
title = "Jetson Nano"
weight = 11
+++

# Set up your device

This lab will help you set up an [nVidia Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) as expected for the remainder of the labs.

While your Nano may have come with the latest version of JetPack (nVidia's SDK) pre-installed, for the purposes of this guide, a fresh installation will be created. If you are uncertain if the SDK has been pre-installed, you can follow this guide.

### STEP 1: Download the SD Card Image for the Latest JetPack

_This guide was prepared with JetPack v. 4.5.1_

1. From the [JetPack SDK Page](https://developer.nvidia.com/embedded/jetpack) find the SD Card Image Link for the Jetson Nano.

2. Click the **Download the SD Card Image** link or use the following commands (Linux):
```bash
cd ~

wget https://developer.nvidia.com/jetson-nano-sd-card-image -O 'image.zip'
```

**NB**- the image is over 6GB and contains the complete OS for the Nano. The time to download this image may be significant depending on your bandwidth.

3. Unzip the image
```bash
cd ~

unzip <zipfile-downloaded-above>
```

This will expand the `sd-blob-b01.img` file.

### STEP 2: Write the image to the SD Card

1. Insert an SD Card of at least 16GB (32GB or more recommended) into a card reader and insert the card reader into the host computer's USB port.

2. Identify the device name for the card
> **Linux**

Type these commands into the terminal:

```bash
 cd ~ # or directory where you extrace the img file

lsblk
# inspect the list to find the disk you just inserted. 
#  You may need to list it first WITHOUT the card and then again to determine the device name.
# 
# The device name will be of the form `/dev/sd<x>` where <x> is a letter.
#
export disk_letter=<insert_your_disk_letter>
```


 > **MacOS**

 Use the Terminal app to enter these commands:

 ```bash
 cd ~ # or directory where you extrace the img file

diskutil list
# inspect the list to find the disk you just inserted. 
#  You may need to list it first WITHOUT the card and then again to determine the device name.
# 
# The device name will be of the form `/dev/disk<n>` where <n> is an integer.
#

export disk_num=<insert_your_disk_number>
```

3. Erase the Card 
_Required for MacOS, optional for Linux hosts_

> **MacOS**

```bash
diskutil eraseDisk ExFAT Untitled /dev/disk$disk_num
diskutil unmountDisk /dev/disk$disk_num
```

4. Write the image file
> **Linux**
```bash
sudo dd if=sd-blob-b01.img of=/dev/rdisk$disk_letter
```

> **MacOS**
```bash
sudo dd if=sd-blob-b01.img of=/dev/rdisk$disk_num
# note use of /dev/rdisk above
```

**NB-** writing the image may take a significant amount of time (upwards of 30 minutes) depending on the speed of your disk, USB interface, Card Reader, and the SD Card itself.

5. Remove the card
```bash

# Linux
diskutil eject /dev/disk$disk_letter

# MacOS
diskutil eject /dev/disk$disk_num
```

Remove the card from the reader.

### STEP 3: Boot and configure the device

1. With the device powered OFF, insert the SD Card into the device

2. Connect a USB cable to the device's micro-USB port and an open USB port on the host **OR** connect and HDMI monitor, keyboard and mouse to the device.

3. Power the device ON

4. _(USB Cable only)_ Open a serial program on the host computer
> **Linux**

Determine the device name for the serial port
```bash
ls /dev/ttyUSB*
# as with the disks, you may need to disconnect, and reconnect to compare available device lists
#  device name will be of the form /dev/ttyUSB<n> where n is an integer
#
export dev_num=<n>
```

Open the serial port -- using `picocom` as example, there are many other programs.

```bash
sudo picocom -b 115200 /dev/ttyUSB$dev_num
```

If the current user is a member of the `dialout` group, `sudo` is not necessary. It is also possible to construct `udev` rules to avoid `sudo` here.

Press the **Return** key and follow the on-screen prompts to set up the device and network connections. Accept all defaults--recommend expanding the filesystem to the maximum.

_The device may disconnect after configuration_

4. _(Monitor and Keyboard)_ Follow the on-screen prompts to set up the device and network connections. Accept all defaults--recommend expanding the filesystem to the maximum

5. Verify SSH connectivity
_(Optional, but recommended)_ This guide assumes you will connect to the device over `ssh`. However, all commands can be entered directly on the device if desired.

From a terminal on the host or other SSH program (e.g. PuTTY on Windows):
```bash
ssh <user>@<hostname>.local
# where hostname and user are the ones you set up in #4 and the device is on the same network as the host

ssh <user>@<ip>
# if zeroconf is not running or other networking arrangement
```

You have now added the device to your network and verified it's connectivity. For the remainder of this workshop, use whatever mechanism you used to connect whenever you see `ssh <user>@<device>`
