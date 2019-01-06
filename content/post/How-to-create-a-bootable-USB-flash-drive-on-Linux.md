+++
title = "How to create a bootable USB flash drive on Linux"
disqus_title="How to create a bootable USB flash drive on Linux"
date = "2019-01-03"
author = "Selwyn Rogers"
cover = ""
description = "A short tutorial on how to use the dd command in Linux to create a bootable USB flash drive."
+++

In this short guide I will explain how to use the handy "dd" command in GNU/Linux to create a bootable
USB flash drive without the need of installing any additional GUI application. This method will also work on
macOS and most of the other POSIX-compliant operating systems.


## dd command-line utility

"dd" is a command-line utility for Unix and Unix-like operating systems that can be used for
various tasks including creating backups, converting and copying files and in our case flashing
drives. "dd" is included in most POSIX-compliant operating systems such as GNU/Linux, FreeBSD and MacOS.


## Download Ubuntu installation image

In this example we will download an installation image of Ubuntu and then flash a USB drive.

First download the installation image from the official Ubuntu website:  https://www.ubuntu.com/#download
As of this writing the current version is 18.10. Feel free to download any other version or an
entirely different GNU/Linux distribution of your liking.

Note: In case the downloaded image is compressed as a zip or tar file, make sure to extract it
beforehand. Depending on the distribution the image file should end with ".iso" or ".img"

For the offical Ubuntu 18.10 image you will need a USB drive with a minimum of 2 GB disk space.


## Check your USB drive's device name.

Before we can start the flashing process, we have to check the exact device name of the USB
drive. This is crucial as we don't want to overwrite any other partition by mistake. Note that all
data will be erased during the process! "dd" can overwrite any partition of your machine
including your primary operating system partition!

### Using fdisk to retrieve device name

"fdisk" is a partition table manipulator that we will now use to list all internal and
external drives including our USB drive. To be safe I recommend unplugin any other external drive
e.g. external hard drives etc.

Open a terminal and type following command to retrieve a list of all drives. You will need
sudo priviliges.

```bash
sudo fdisk -l
```

The output should look like the example bellow:

```bash
Disk /dev/sdb: 7.6 GiB, 8178892800 bytes, 15974400 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0009ebfa
```


Go through the list and identify the appropiate USB drive.
Write down the device name which follows the naming pattern of "/dev/sdX"
(Change "X" accordingly)

## Flashing the drive

Now that we obtained the correct device name of our USB drive and downloaded the
Ubuntu installation image, we finally can start the flashing process.
Replace "/dev/sdX" and the path of the image file accordingly.

Make sure that the device name is the name of the whole USB drive e.g. "/dev/sdb" without
any appending number. For example: In the case of a
partition called "/dev/sdb1" the device name would be "/dev/sdb" without the "1"

We will use a block size ("bs=4M") of 4 Megabytes - you can reduce the block size e.g. to
"1M" if there should be any issues. Reducing block size will prolong the flashing process.

To track the progress we will add an "status" argument with the value of "progress"
If you run an older "dd" version this might not work and you won't get an progress output.

Note: Depending on the image size and defined block size the process can take up to 5 minutes or
longer. Be patient and do not abort the process even though it might look as if it has frozen.


```bash
sudo dd bs=4M if=/path_of_image/ubuntu-18.10-desktop-amd64.iso of=/dev/sdX status=progress
conv=fsync
```

After the flashing process has finished unmount your USB drive, reboot your machine and if
everything wen't right, you should be able to able to boot from the USB drive.
