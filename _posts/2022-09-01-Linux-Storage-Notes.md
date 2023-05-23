---
layout: post
title: "Linux Storage Notes"
date: 2022-09-01 10:00:00 +0900
categories: [linux, storage]
tags: [storage, ssd, hdd, disks, linux, lpic, notes]
---

A collection of rough notes around managing storage in Linux


## 1. Drive sanitization

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

