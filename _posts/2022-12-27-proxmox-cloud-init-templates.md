---
layout: post
title: "Proxmox cloud-init templates"
date: 2022-12-27 10:00:00 +0900
categories: [proxmox]
tags: [template, cloud-init, vm, linux]
comments: true
---
# Introduction and why 

TODO

# Download cloud templates from Debian and Ubuntu

Listed below are the official download links for Debian and Ubuntu release

- Debian 11 Bullseye:<br>
`wget https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-genericcloud-amd64.qcow2`
- Debian 11 Bullseye (Backports):<br>
`wget https://cloud.debian.org/images/cloud/bullseye-backports/latest/debian-11-backports-generic-amd64.qcow2`
- Debian 10 Buster:<br>
`wget https://cloud.debian.org/images/cloud/buster/latest/debian-10-genericcloud-amd64.qcow2`
- Ubuntu Server 20.04 LTS (Focal Fossa):<br>
`wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img`
- Ubuntu 18.04 LTS (Bionic Beaver):<br>
`wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img`

## Confirm the checksum status

- To confirm the checksum of an image with Debian below is an example. 
  ```shell
  echo $(curl https://cloud.debian.org/images/cloud/bullseye/latest/SHA512SUMS | grep debian-11-genericcloud-amd64.qcow2 |  awk '{ print $1 }' ) debian-11-genericcloud-amd64.qcow2 | sha512sum --check
  ```

# Creating your first VM template with a cloud image. 

- Setup the initial variables. 
  ```shell
  VM_TEMPLATE_ID=9001
  VM_PASSWORD=supercrect  # < Clear text input
  VM_USER=karubits
  TEMPLATE_NAME=debian-cloud-template
  CLOUD_IMAGE=focal-server-cloudimg-amd64.img
  STORAGE=nvme-2tb
  ```
- Create the initial template with the minimum values. 
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
- Add the virtual disk controller and attached the imported disk to the template. 
  ```shell
  qm set $VM_TEMPLATE_ID \
    --scsihw virtio-scsi-pci \
    --scsi0 $STORAGE:vm-$VM_TEMPLATE_ID-disk-0,discard=on,ssd=1 \
    --boot c \
    --bootdisk scsi0
  ```

- Configure the default values for your cloud-init template. This can also be done on the Promxox UI:
  ```shell
  qm set $VM_TEMPLATE_ID \
    --nameserver="10.7.7.3 10.7.7.2" \
    --searchdomain=core.io \
    --ipconfig0=ip=dhcp \
    --ciuser=$VM_USER \
    --cipassword=$VM_PASSWORD
  ```

- Lastly, convert the VM into a template with the following command:<br>
  ```shell
  qm template $VM_TEMPLATE_ID
  ```

# Create a new VM from the template for testing

- Before cloning the VM I use a variable to find the next available VM ID as the clone command requires you to set an ID. Then take a full a clone of the new template. 
  ```shell
  NEXT_VM=$(pvesh get /cluster/nextid)

  qm clone $VM_TEMPLATE_ID $NEXT_VM \
    --full=true \
    --name=wow-so-quick-to-deploy
  ```
- As the default image size is very small (247mb for Debian), expand the disk to make the VM useful and then start the VM:
  ```shell
  qm resize $NEXT_VM scsi0 +15G
  qm start $NEXT_VM
  ```
- Your new VM from your template should now be starting. with all your new setting set


# Links
- [Debian Official Cloud Images](https://cloud.debian.org/images/cloud/)
- [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/)
- [Alpine Linux Cloud Images](https://alpinelinux.org/cloud/) <br> Note: Currently NoCloud is not supported but coming soon. 

# References
1. *[Proxmox Wiki - Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)*
2. *[Proxmox VE - How to build an Ubuntu 22.04 Template (Updated Method) - Video](https://www.youtube.com/watch?v=MJgIm03Jxdo)*
3. *[Proxmox VE – How to build an Ubuntu 22.04 Template (Updated Method) - Blog](https://www.learnlinux.tv/proxmox-ve-how-to-build-an-ubuntu-22-04-template-updated-method/)*