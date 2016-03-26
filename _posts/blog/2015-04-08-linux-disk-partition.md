---
layout: post
title: "Linux disk partition"
categories: blog
---
### fdisk
handle disk which is less than 2T

### parted
1. partition: `sudo parted /dev/sde mkpart`
- make file system: `sudo mkfs.ext4 /dev/sde1`

### mount
1. `sudo dumpe2fs -h /dev/sde1`, get the partition label
- `sudo vim /etc/fstab`
- add `UUID=8f1094db-3539-4c38-9184-789c809b3730       /mnt/hive       ext4    defaults        0       2` or `/dev/sde1/ /mnt/hive ext4 defaults 0 2`
