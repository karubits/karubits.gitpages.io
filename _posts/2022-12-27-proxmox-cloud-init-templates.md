---
layout: post
title: "Proxmox cloud-init templates"
date: 2022-12-27 10:00:00 +0900
categories: [proxmox]
tags: [template, cloud-init, vm, linux]
---
# Introduction and why

Using cloud images provided by the major distribution creators over traditional installation media offers several advantages in the context of cloud computing and virtualization environments. Here are some of the key benefits:

- Optimized for Cloud Environments: Cloud images are specifically designed and optimized for running in virtualized or cloud environments. They come with preconfigured settings, drivers, and components tailored to these environments, ensuring smooth performance and compatibility.
- Smaller Footprint: Cloud images are usually stripped down to include only essential packages, reducing the overall size of the image. This is important for faster deployment, as well as reducing storage costs in cloud environments.
- Faster Deployment: Cloud images are ready to use out of the box, so you can quickly deploy instances without the need for a full installation process. This reduces setup time and makes it easier to scale your infrastructure.
- Automated Configuration: Cloud images often include tools and scripts for automated configuration and deployment. This can streamline the process of setting up networking, security settings, and other parameters.
- Regular Updates: Cloud images are regularly updated to include the latest security patches and software updates. This helps ensure that your instances are running the most up-to-date and secure software.
- Integration with Cloud Services: Cloud images often come with cloud-init, a tool that enables you to customize and configure instances during the first boot. This allows for seamless integration with cloud platforms' services and APIs.
- Image Reusability: Once you've configured and customized a cloud image, you can save it as a custom image or snapshot. This allows you to create new instances from the customized image, saving you time and ensuring consistency across deployments.
- Version Consistency: Using cloud images helps maintain consistency across different instances. This is especially important when managing a large number of virtual machines, as manual installations might introduce inconsistencies.
- Rapid Scaling: Cloud images make it easier to scale your infrastructure up or down based on demand. You can quickly launch new instances without going through a time-consuming installation process.
- Resource Efficiency: Cloud images are optimized to run efficiently in virtualized environments, making better use of resources like CPU, memory, and storage.


## clout-init

cloud-init is a versatile package used in cloud computing environments to streamline the setup and configuration of virtual machine instances. It gathers instance-specific information from metadata services and user-provided data during instance launch. Using a modular architecture and YAML-based configuration, cloud-init automates tasks such as network setup, user account creation, and software installation. It allows users to define custom actions and settings through user data, ensuring efficient, consistent, and customizable initialization of instances while promoting compatibility across different Linux distributions and cloud platforms.

With Proxmox cloud-init is used to automatically setup IP addressing, name servers, ssh keys, and the initial password. 

# Download cloud templates of your preferred distributions

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

Or download most of them in one shot with the following bash script. 

```bash
#!/bin/bash

# Set variables
TMP_DIR="/tmp"

# Define arrays for image names, URLs, checksum URLs, and expected checksum algorithms
images=(
  "Debian 12"
  "Debian 11" 
  "Ubuntu Focal" 
  "Ubuntu Bionic"  
  "Kali Linux"
)

urls=(
    "https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2"
    "https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-genericcloud-amd64.qcow2"
    "https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img"
    "https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img"
    "http://kali.download/cloud-images/current/kali-linux-2023.2-cloud-genericcloud-amd64.tar.xz"
)

checksum_urls=(
    "https://cloud.debian.org/images/cloud/bookworm/latest/SHA512SUMS"
    "https://cloud.debian.org/images/cloud/bullseye/latest/SHA512SUMS"
    "https://cloud-images.ubuntu.com/focal/current/SHA256SUMS"
    "https://cloud-images.ubuntu.com/bionic/current/SHA256SUMS"
    "https://kali.download/cloud-images/current/SHA256SUMS"
)

checksum_algorithms=(
    "sha512"
    "sha512"
    "sha256"
    "sha256"
    "sha256"
)

# Navigate to the temporary directory
cd "$TMP_DIR"

# Iterate through the arrays
for i in "${!images[@]}"; do
    echo "🔹 Now Downloading ⏬ ${images[$i]}..."
    
    # Download the image
    wget "${urls[$i]}"
    
    # Calculate and compare checksums if a checksum URL is provided
    if [ ! -z "${checksum_urls[$i]}" ]; then
        image_file=$(basename "${urls[$i]}")
        checksum_url="${checksum_urls[$i]}"
        expected_algorithm="${checksum_algorithms[$i]}"
        
        actual_sum=$(openssl dgst -"$expected_algorithm" "$image_file" | awk '{ print $2 }')
        expected_sum=$(curl -s "$checksum_url" | grep "$image_file" | awk '{ print $1 }')
        
        if [ "$actual_sum" = "$expected_sum" ]; then
            echo "🔹 Checksums match for $image_file. ✅"
            echo ""
        else
            echo "🔹 Checksums do not match ❌. The file $image_file might have been corrupted."
        fi
    fi
done
```

# Creating your first VM template with a cloud image. 


- [Optional] Install apt packages into the images, I like to have the qemu-guest-agent installed by default. 
   ```shell
   # If you haven't already you will need to install libguestfs-tools for adding packages to a disk image. 
   sudo apt install --no-install-recommends --no-install-suggests libguestfs-tools -y
   
   # Then add the required packages before importing the image. 
   sudo virt-customize -a $CLOUD_IMAGE --install qemu-guest-agent,lnav,ca-certificates,apt-transport-https,net-tools,dnsutils
   ```


> Its generally not good practice to install packages on your hypervisors unless absolutely necessary. Downloads, checksum checking, and installing extra packages can be done on a seperate PC and the final disk image uplaoded to the hypervisor using SCP. 
{: .prompt-warning }

- On proxmox start by settings variables to make the steps more generic and repeatable.  
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
    --nameserver="10.7.7.3 10.7.7.2" \
    --searchdomain=core.io \
    --ipconfig0=ip=dhcp \
    --ciuser=$VM_USER \
    --cipassword=$VM_PASSWORD
  ```

- Lastly, convert the VM into a template with the following command.
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


# References
1. *[Proxmox Wiki - Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)*
2. *[Proxmox VE - How to build an Ubuntu 22.04 Template (Updated Method) - Video](https://www.youtube.com/watch?v=MJgIm03Jxdo)*
3. *[Proxmox VE – How to build an Ubuntu 22.04 Template (Updated Method) - Blog](https://www.learnlinux.tv/proxmox-ve-how-to-build-an-ubuntu-22-04-template-updated-method/)*