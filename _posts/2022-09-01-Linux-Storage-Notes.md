---
layout: post
title: "Linux Storage Notes"
date: 2022-09-01 10:00:00 +0900
categories: [linux, storage]
tags: [storage, ssd, hdd, disks, linux, lpic, notes]
---

A collection of rough notes around managing storage in Linux

## 1. Partitioning and mounting a new drive (ext4)

1. Locate the new drive you want to format. `lsblk --fs`, in this example the new drive is /dev/sdb.
2. Create a partition label (usually mbr or gpt)
   ```bash
   sudo parted /dev/sdb mklabel gpt
   ```
3. Partition 100% of the drive as ext4
   ```bash
   sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
   ```
4. Label the parition (in this case "data" is the label) 
   ```bash
   mkfs.ext4 -L data /dev/sdb1
   ```
5. Create a directory to mount to. `mkdir /data`
6. Add an entry in the /etc/fstab to the new partition is mounted on every reboot. 
   ```
   # /etc/fstab
   LABEL=data /data ext4 rw,discard,errors=remount-ro,x-systemd.growfs 0 1
   ```
7. reread the fstab to mount the new mount. 
   ```
   sudo systemctl daemon-reload
   sudo mount -a
   ```
8. Confirm the partition is mounted. 
   ```shell
   root@server02:/etc/network# lsblk --fs
   NAME    FSTYPE  FSVER LABEL  UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
   sdb                                                                              
   └─sdb1  ext4    1.0   data   a55217b4-f3eb-44fc-98a1-04d65158a203  466.1G     0% /data
   ```

## 2. Online Resize a vDisk that (ext4)

- This article assumes the vdisk has already been resized on the hypervisor side. 
- In this particular case we have a secondary vdisk used for data. The drive is mounted under /mnt/data. 
While resizing the disk, to avoid any potential corruption the disk should be unmounted. 
   ```bash
   umount /mnt/data
   ```
- If not installed already, install the parted package. `apt install -y parted`
- Then resize the partition. When promted with a Warning, type in Fix to ensure all space will be available for the partition. 
   ```bash
   root@server:~# parted /dev/sdc resizepart 1
   Warning: Not all of the space available to /dev/sdc appears to be used, you can fix the GPT to use all of the space (an extra
   2097152000 blocks) or continue with the current setting? 
   Fix/Ignore? fix                                                     
   Warning: Partition /dev/sdc1 is being used. Are you sure you want to continue?
   Yes/No? yes                                                               
   End?  [2147GB]?                                                           
   Information: You may need to update /etc/fstab.
   ```
- Then resize the file system.
   ```bash
   root@server:~# resize2fs /dev/sdc1                                         
   resize2fs 1.47.0 (5-Feb-2023)
   Resizing the filesystem on /dev/sdc1 to 786431739 (4k) blocks.
   The filesystem on /dev/sdc1 is now 786431739 (4k) blocks long.
   ```
- To confirm the size of the partition `df` can be used. To confirm the parition `lsblk' can be used. 
   ```bash
   root@server:~# df -h
   Filesystem      Size  Used Avail Use% Mounted on
   /dev/sdc1       2.0T  2.0T     0 100% /mnt/data
   
   root@server:~# lsblk
   NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   └─sdc1    8:17   0     2T  0 part /mnt/data
   ```
  - Assuming this disk is already in the `/etc/fstab` then remount the partition by typing in `mount -a`
 
  
## 3. Drive sanitization

[NIST Publiciation 800-88 Rev 1 - Guidelines for Media Sanitization](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-88r1.pdf)

### 1.1 Secure Erase for NVME Drive


1. Install the nvme-cli tools \
`sudo apt install -y nvme-cli`
2. confirm which disk to target  \
`lsblk -o name,model,serial,type | grep nvme | grep disk` or \
`nvme list`
3. Perform a crytographic erase \
 `sudo nvme format --ses=2 /dev/nvme2n1`

| Value | Definition |
| :- | :-
| 1 | 	User Data Erase: All user data shall be erased, contents of the user data after the erase is indeterminate (e.g., the user data may be zero filled, one filled, etc). The controller may perform a cryptographic erase when a User Data Erase is requested if all user data is encrypted. |
| 2 | Cryptographic Erase: All user data shall be erased cryptographically. This is accomplished by deleting the encryption key.|


> If you see the follow: \
NVMe status: INVALID_FORMAT: The LBA Format specified is not supported. This may be due to various conditions(0x410a) \
Put the PC to sleep/suspend and wake it backup and rerun the command. 
{: .prompt-tip }

### 1.2 Secure Erase a SATA Drive

1. Install hdparm \
`sudo apt install -y hdparm`
2. Identify the disk to format. \
`lsblk -o name,model,serial,type`
3. Ensure the SATA Drive supports security erase enchanced mode and the disk is "not frozen". If the disk says frozen then put the PC to sleep and wake it up again. \
`sudo hdparm -I /dev/sdX`
4. Secure erase the disk
   ```shell
   sudo hdparm --user-master u --security-erase-enhanced password /dev/sdX
   # If enchanced security erase is not supported:
   sudo hdparm --user-master u --security-set-pass password /dev/sdX
   ```


   
