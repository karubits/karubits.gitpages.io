---
layout: post
title: "Firmware upgrade Intel DC SSD on Linux"
date: 2022-12-21 10:00:00 +0900
categories: [server]
tags: [server. dell, intel, proxmox, firmware]
---


# Server Preperation / Hardware Notes


## Intel Datacenter SSD Firmware

> At the time of writting, Intel's NAND SSD business has been acquired by Solidigm. The tooling which was previously called Intel MAS is no longed supported. 
{: .prompt-tip }

This example will cover upgrading the firwmare of Intel DC series firmware on a Debian operating system. This same steps are applicable to Proxmox hypervisors as well. 

This example will cover upgrade the Intel DC P4510 and the same steps will cover other Intel DC series firmware as well.

### Links, Guides, and Downloads

- Download the latest SST package from Solidigm
[here](https://www.solidigm.com/us/en/support-page/drivers-downloads/ka-00085.html)

- For advanced usuage of the SST tool you can refer to the CLI User Guide [here](https://sdmsdfwdriver.blob.core.windows.net/files/kba-gcc/drivers-downloads/ka-00085--sst/sst--1-4/sst-cli-user-guide-public-727329-005us.pdf). 
- At the time of writing v1.4 was the latest. 

### Procedure

- Download the SST tool directly on the server (v1.4 link below). <br>
    `wget https://sdmsdfwdriver.blob.core.windows.net/files/kba-gcc/drivers-downloads/ka-00085--sst/sst--1-4/sst-cli-linux-deb--1-4.zip`
- Exact the package <br>
    `unzip sst-cli-linux-deb--1-4.zip`
- Install the SST CLI tool (Debian x64) <br>
    `dpkg -i sst_1.4.221-0_amd64.deb`
- View available SSDs and confirm which ones can be upgraded. <br>
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
- Note the Index number if you can see a new update listed in the 'FirmwareUpdateAvailable' field. 
- Trigger the upgrade <br>
    ```bash
    sst load -force -ssd 0

        Checking for firmware update...

    - Intel SSD DC P4510 Series PHLJ0373032N2P0BGN  -

    Status : Firmware updated successfully. Please  reboot the system.
    ```
- Then reboot the system for the new firmware to take effect. 

