---
layout: post
title: "Proxmox cloud-init templates"
date: 2022-12-27 10:00:00 +0900
categories: [proxmox]
tags: [template, cloud-init, vm, linux, proxmox]
---
Automate virtual machine provisioning with cloud templates and cloud init with proxmox. 

## Introduction

Cloud images are optimized images of a Linux distribution that are design to run in cloud environments. There are generally updates faster then the ISO images, and are slimmed down with some built in support for automated provisioning using cloud init. 


## Download links to cloud templates

Listed below are the official download links for some of the more common cloud images. 

| Distro | Release | Download | Checksum
| :-- | :-- | :--: | :-- |
| Debain | 12 Bookworm | [💿](https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2) | [🔑](https://cloud.debian.org/images/cloud/bookworm/latest/SHA512SUMS)
| Debian | 11 Bullseye (Backports) | [💿](https://cloud.debian.org/images/cloud/bullseye-backports/latest/debian-11-backports-genericcloud-amd64.qcow2) | [🔑](https://cloud.debian.org/images/cloud/bullseye-backports/latest/SHA512SUMS)
| Debian | 11 Bullseye |  [💿](https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-genericcloud-amd64.qcow2) | [🔑](https://cloud.debian.org/images/cloud/bullseye/latest/SHA512SUMS)
| Debian | 10 Buster |  [💿](https://cloud.debian.org/images/cloud/buster/latest/debian-10-genericcloud-amd64.qcow2) | [🔑](https://cloud.debian.org/images/cloud/buster/latest/SHA512SUMS)
| Ubuntu | 20.04 LTS Focal Fossa | [💿](https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img) | [🔑](https://cloud-images.ubuntu.com/focal/current/SHA256SUMS)
| Ubuntu | 18.04 LTS Bionic Beaver | [💿](https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img) | [🔑](https://cloud-images.ubuntu.com/bionic/current/SHA256SUMS)
| Rocky | Linux 9 | [💿](https://dl.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud-Base.latest.x86_64.qcow2)| [🔑](https://dl.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud-Base.latest.x86_64.qcow2.CHECKSUM)
| Rocky | Linux 8 | [💿](https://dl.rockylinux.org/pub/rocky/8/images/x86_64/Rocky-8-GenericCloud-Base.latest.x86_64.qcow2)| [🔑](https://dl.rockylinux.org/pub/rocky/8/images/x86_64/Rocky-8-GenericCloud-Base.latest.x86_64.qcow2.CHECKSUM)
| Alama | Linux 8 | [💿](https://repo.almalinux.org/almalinux/8/cloud/x86_64/images/AlmaLinux-8-GenericCloud-latest.x86_64.qcow2) | [🔑](https://repo.almalinux.org/almalinux/8/cloud/x86_64/images/CHECKSUM) 
| Alama | Linux 9 | [💿](https://repo.almalinux.org/almalinux/9/cloud/x86_64/images/AlmaLinux-9-GenericCloud-latest.x86_64.qcow2) | [🔑](https://repo.almalinux.org/almalinux/9/cloud/x86_64/images/CHECKSUM) 
| Kali | 2022.4 | [💿](https://kali.download/cloud-images/current/kali-linux-2022.4-cloud-genericcloud-amd64.tar.xz) | [🔑](https://kali.download/cloud-images/current/SHA256SUMS)
| Fedora | 38 | [💿](https://ftp.riken.jp/Linux/fedora/releases/38/Cloud/x86_64/images/Fedora-Cloud-Base-38-1.6.x86_64.qcow2 ) | [🔑](https://ftp.riken.jp/Linux/fedora/releases/38/Cloud/x86_64/images/Fedora-Cloud-38-1.6-x86_64-CHECKSUM)

<br>

## Creating your first VM template with a cloud image. 

> Its generally not good practice to install packages on your hypervisor. Downloads, checksum checking, and installing extra packages can be done on a separate PC and the final disk image can then be transferred to the hypervisor using SCP. 
{: .prompt-warning }

- Download one of the distributions cloud images. Example below:
   ```shell
   wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
   ```
- [Optional] I prefer my images to have a few essential packages installed, especially the qemu-guest-agent that is used by Proxmox. To do so you will need `libguestfs-tools` installed. 
   ```shell
   sudo apt install --no-install-recommends --no-install-suggests libguestfs-tools -y
   ```
- Then use `virt-cutomize` to install your preferred packages. 
   ```shell
   # Then add the required packages before importing the image. 
   CLOUD_IMAGE=debian-12-generic-amd64.qcow2

   sudo virt-customize -a $CLOUD_IMAGE --install qemu-guest-agent,lnav,ca-certificates,apt-transport-https,net-tools,dnsutils
   ```
- If you have perform the above on a seperate PC then scp the modified disk image to the target hypervisor. 
- On proxmox start by setting variables to make the steps more generic and repeatable.  
  ```shell
  CLOUD_IMAGE=focal-server-cloudimg-amd64.img
  DNS1=1.1.1.1
  DNS2=8.8.8.8
  DNSSEARCH=karubits.com
  STORAGE=nvme-2tb
  TEMPLATE_NAME=debian-12-cloud-template
  VM_TEMPLATE_ID=9001
  VM_USER=karubits
  
  # Using read prevents the password from been saved in bash history
  read -rs -p "Enter password: " VM_PASSWORD
  ```
- Create the initial template with the minimum values. You can adjust these to suit your environment.  
  ```shell
  qm create $VM_TEMPLATE_ID \
    --name $TEMPLATE_NAME \
    --core 2 \
    --memory 2048 \
    --net0 virtio,bridge=vmbr0 \
    --ide2 $STORAGE:cloudinit \
    --serial0 socket \
    --vga serial0 \
    --onboot 1 \
    --agent 1,fstrim_cloned_disks=1 \
    --tablet 0 \
    --ostype l26
  ```
- Import the downloaded disk image into the new VM template. 
  ```shell
  qm importdisk $VM_TEMPLATE_ID \
      $CLOUD_IMAGE \
      $STORAGE
  ```
- Add the virtual disk controller and attache the imported disk to the template. 
  ```shell
  qm set $VM_TEMPLATE_ID \
    --scsihw virtio-scsi-pci \
    --scsi0 $STORAGE:vm-$VM_TEMPLATE_ID-disk-0,discard=on,ssd=1 \
    --boot c \
    --bootdisk scsi0
  ```

- Configure the default values for your cloud-init template. (This can also be done on the Promxox UI)
  ```shell
  qm set $VM_TEMPLATE_ID \
    --nameserver="$DNS1 $DNS2" \
    --searchdomain=$DNSSEARCH \
    --ipconfig0=ip=dhcp \
    --ciuser=$VM_USER \
    --cipassword=$VM_PASSWORD
  ```

- Lastly, convert the VM into a template with the following command.
  ```shell
  qm template $VM_TEMPLATE_ID
  ```

## Create a new VM from the template for testing

- Before cloning the VM I use a variable to find the next available VM ID as the clone command requires you to set an ID. Then take a full a clone of the new template. 
  ```shell
  NEXT_VM=$(pvesh get /cluster/nextid)

  qm clone $VM_TEMPLATE_ID $NEXT_VM \
    --full=true \
    --name=wow-so-quick-to-deploy
  ```
- As the default image size is very small (e.g. 247mb for Debian), expand the disk to make the VM useful and then start the VM:
  ```shell
  qm resize $NEXT_VM scsi0 +15G
  qm start $NEXT_VM
  ```



## References
1. *[Proxmox Wiki - Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)*
2. *[Proxmox VE - How to build an Ubuntu 22.04 Template (Updated Method) - Video](https://www.youtube.com/watch?v=MJgIm03Jxdo)*
3. *[Proxmox VE – How to build an Ubuntu 22.04 Template (Updated Method) - Blog](https://www.learnlinux.tv/proxmox-ve-how-to-build-an-ubuntu-22-04-template-updated-method/)*