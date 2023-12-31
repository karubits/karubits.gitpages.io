---
layout: post
title: "Firmware upgrade Intel DC SSD on Linux"
date: 2022-12-21 10:00:00 +0900
categories: [server]
tags: [server. dell, intel, proxmox, firmware]
---


# Server Preparation / Hardware Notes


## Intel Datacenter SSD Firmware

> At the time of writing, Intel's NAND SSD business has been acquired by Solidigm. The tooling which was previously called Intel MAS. If you have Intel Optane drives, please continue to use Intel MAS. Both tools can be install in parallel. 
{: .prompt-tip }

This guide covers upgrading the firmware on Intel DC series SSDs on a Debian based operating system. This same steps are applicable to Proxmox hypervisors as well, although when CEPHs is running and the target disk is used as an OSD I found the drive to be locked and I have not found a way to upgrade the firmware without using an external system. 

### Links, Guides, and Downloads

- Download the latest SST package from Solidigm
[here](https://www.solidigm.com/us/en/support-page/drivers-downloads/ka-00085.html)
- Intel's MAS Firmware tool for Optane drives can be found [here](https://www.intel.com/content/www/us/en/download/19520/intel-memory-and-storage-tool-cli-command-line-interface.html?v=t)
- For advanced usuage of the SST tool you can refer to the CLI User Guide [here](https://sdmsdfwdriver.blob.core.windows.net/files/kba-gcc/drivers-downloads/ka-00085--sst/sst--1-4/sst-cli-user-guide-public-727329-005us.pdf). 
- At the time of writing v1.11 was the latest for the SST tool from Solidigm.

### Procedure

1. Go to Solidigm website to download the latest Solidigm Storage Tool (SST) at the link [here](https://www.solidigmtechnology.jp/support-page/drivers-downloads/ka-00085.html). <br>
Alternatively you can download the SST tool directly from the link below (v1.11 as of December 2023). <br>
    `wget https://sdmsdfwdriver.blob.core.windows.net/files/kba-gcc/drivers-downloads/ka-00085/sst--1-11/sst-cli-linux-deb--1-11.zip`
2. Exact the package <br>
    `unzip sst-cli-linux-deb--1-11.zip`
3. Install the SST CLI tool (Debian x64) <br>
    `dpkg -i sst_1.11.268-0_amd64.deb`
4. View available SSDs and confirm which ones can be upgraded. <br>
    ```bash
    sst show -ssd

    - PHLJ0373032N2P0BGN 1 -

    Bootloader : 0203
    Capacity : 2.00 TB (2,000,398,934,016 bytes)
    DevicePath : /dev/nvme0n1
    DeviceStatus : Healthy
    Firmware : VDV10131
    FirmwareUpdateAvailable : Firmware=VDV10184     Bootloader=VB1B0181
    Index : 0
    MaximumLBA : 3907029167
    ModelNumber : INTEL SSDPE2KX020T8
    NamespaceId : 1
    PercentOverProvisioned : 100.00
    ProductFamily : Intel SSD DC P4510 Series
    SMARTEnabled : True
    SectorDataSize : 512
    SerialNumber : PHLJ0373032N2P0BGN
    ```
5. Note the Index number if you can see a new update listed in the 'FirmwareUpdateAvailable' field. 
6. Trigger the upgrade <br>
    ```bash
    sst load -force -ssd 0

        Checking for firmware update...

    - Intel SSD DC P4510 Series PHLJ0373032N2P0BGN  -

    Status : Firmware updated successfully. Please  reboot the system.
    ```
7. Reboot the system for the new firmware to take effect. 


> If your SSDs are behind a Megaraid controller (including Dell PERC) you won't be able to see the SSDs without enabling the following command. <br>
`sst set –system EnableLSIAdapter=True`
{: .prompt-tip }

