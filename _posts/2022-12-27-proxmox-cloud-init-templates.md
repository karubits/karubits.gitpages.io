---
layout: post
title: "Proxmox cloud-init templates"
date: 2022-12-27 10:00:00 +0900
categories: [proxmox]
tags: [template, cloud-init, vm, linux]
comments: true
---

## Download cloud templates from Debian and Ubuntu

- The latest Debian cloud images for virtual machines can be downloaded with the following commands:
```shell
cd /tmp

# Debian 11 Bullseye
`wget https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-genericcloud-amd64.qcow2`

# Confirm the checksum status

```shell
echo $(curl https://cloud.debian.org/images/cloud/bullseye/latest/SHA512SUMS | grep debian-11-genericcloud-amd64.qcow2 |  awk '{ print $1 }' ) debian-11-genericcloud-amd64.qcow2 | sha512sum --check 
```
- Ubuntu's official image can be downloaded with this URL:
`wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img`


## Creating your first VM template with a cloud image. 

- Setup the initial variables. 
    ```shell
    VM_TEMPLATE_ID=9001
    VM_PASSWORD=supercrect  # < Clear text input
    VM_USER=karubits
    TEMPLATE_NAME=debian-cloud-template
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
        --agent 1 \
        --tablet 0
    ```
- Import the downloaded disk image into the new VM template. 
    ```shell
    qm importdisk $VM_TEMPLATE_ID \
        focal-server-cloudimg-amd64.img \
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
    `qm template $VM_TEMPLATE_ID`

## Create a new VM from the template for testing

- Before cloning the VM I use a varible to find the next available VM ID as the clone command requires you to set an ID. Then take a full a clone of the new template. 
    ```
    NEXT_VM_TEMPLATE_ID=$(pvesh get /cluster/nextid)

    qm clone $VM_TEMPLATE_ID $NEXT_VM_TEMPLATE_ID \
        --full=true \
        --name=fuckyeah3
    ```
- As the default image size is very small (247mb for Debian), expand the disk to make the VM useful and then start the VM:
    ```shell
    qm resize $NEXT_VM_TEMPLATE_ID scsi0 +15G
    qm start $NEXT_VM_TEMPLATE_ID
    ```
- Your new VM from your template should now be starting. with all your new setting set