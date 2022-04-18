---
title: Creating a filesystem and a partition on an EBS volume
date: 2022-04-17 17:08:47 +07:00
tags: [aws]
description: Creating a filesystem and a partition on an EBS volume 
---

Often times, we may have to add additional disks to store application or other data. To make the newly added volume usuable, a filesystem and a partition (if needed) needs to be created from the OS level. In this blog, we will explore the steps to create a partition and a filesystem once a new volume is attached to an EC2 Linux instance.

Execute the following command to check if a filesystem exists on a volume:

* sudo file -s /dev/xvdf

Note: the device label could be different in your case if a different label name was selected while attaching the volume.

If the output of the above command is */dev/xvdf: data*, then there is no file system on the device.

Refer to the following link for the steps to create a filesystem on a raw volume (without partition).

* [Make an Amazon EBS volume available for use on Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)

#### The steps described below creates a partition on the newly added volume and configures an ext4 file system.

1. Check the filesystem on the root volume using *lsblk -f* or *df -hT* command.
2. Switch to the root user - *sudo su -*
3. parted /dev/xvdf
	* print
	* mklabel gpt
	* mkpart primary ext4 1MB 18GB
4. mkfs.ext4 /dev/xvdf1
5. mount /dev/xvdf1 /mnt
6. Check the UUID of the new disk with the *lsblk -f* command.
7. Update the fstab file located in /etc directory.
	* vi /etc/fstab
	* UUID= /mountpoint			  ext4	  defaults,nofail 0 0
	* Note - Append the UUID of the new disk retrieved from step 6 after the *=* sign and update the mountpoint based on the mountpoint set on step 5.
8. *mount -a*
9. *partprobe*

Once the above commands are executed, a persistent parition on a new volume with a filesystem is created.
