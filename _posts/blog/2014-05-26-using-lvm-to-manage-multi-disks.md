---
layout: post
title: "Using lvm to manage multi-disks"
categories: blog
---

Install
----------
* insert disks in your free disk slots
* `sudo apt-get install lvm2`

Create lvm partiton
----------
- `fdisk -l`
- `fdisk /dev/sdb` then follow the default configuration to create partition
- `pvcreate /dev/sdb`
- `vgcreate vg0 /dev/sdb1 /dev/sdc1 /dev/sdd1`
- `pvdisplay` `vgdisplay`
- `lvcreate -L 2T -n data-lvm1 vg0`
- `mkfs.ext4 /dev/mapper/vg0-data--lvm1`

Mount lvm partition
-----------
- `mkdir /data-lvm1`
- `mount /dev/vg0/data-lvm1 /data-lvm1`
- `sudo blkid -o value -s UUID /dev/mapper/vg0-data--lvm1` get logical volumn's UUID
- insert `UUID=d5d184a6-7356-49d1-8f6a-6d33c719479f /data-lvm ext4    defaults    0   2` to `/etc/fstab`


Other memos
----------
* resize extened partition `resize2fs /dev/mapper/vg0-data--lvm1`
* `partprobe`
