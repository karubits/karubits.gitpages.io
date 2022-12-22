---
layout: post
title: "Server Firmware Notes"
date: 2022-08-21 10:00:00 +0900
categories: [server]
tags: [server. dell, mellanox, firmware]
comments: true
---

# Server Preperation / Hardware Notes

## Dell Server

### iDrac - racdm notes

iDrac tools includes a executable called racadm for managing the idrac controller on Dell Servers. On the latest version of iDRAC tools for Linux they have also included Ubuntu (Debian works) support.

Direct Download Links posted below:
- [Dell EMC iDRAC Tools for Linux, v10.1.0.0](https://dl.dell.com/FOLDER07423496M/1/DellEMC-iDRACTools-Web-LX-10.1.0.0-4566_A00.tar.gz)
- [Dell EMC iDRAC Tools for Windows Server(R), v10.2.0.0](https://dl.dell.com/FOLDER07549599M/1/DellEMC-iDRACTools-Web-WINX64-10.2.0.0-4583_A00.exe)


Listed below is an example of a batch file for making various changes to the idrac configuration. 
```batch
# Upgrade to latest iDrac FW before starting
 
SET racadmPath="c:\Program Files\Dell\SysMgt\iDRACTools\racadm\"
SET idracTarget=%1
 
REM # View and backup your license (Note: I found white lines added to the xml. If you need to import you will have to remove)
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn license view
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn license export -c iDRAC.Embedded.1
 
REM # Factory Reset (You 'may' lose your license, also iDrac sets a default static IP. You will need to go to the console and set it back to dhcp via F2 from the bios prompt)
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn racresetcfg
 
REM # Set DNS from DHCP and register the DNS Name
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.IPv4.DNSFromDHCP 1
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Nic.DNSRegister 1
 
REM # Set console to HTML5 by default
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.VirtualConsole.PluginType HTML5
 
REM # Disable telnet
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.Telnet.Enable 0
 
REM # Disable default password warning
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Tuning.DefaultCredentialWarning Disabled
 
REM # Set correct time
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.time.timezone Japan
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.NTPConfigGroup.ntp1 10.2.25.2
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.NTPConfigGroup.ntp2 10.2.25.3
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.NTPConfigGroup.ntp3 ntp.nict.jp
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set idrac.NTPConfigGroup.NTPEnable 1
 
REM # Generate SSL
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn config -g cfgRacSecurity -o cfgRacSecCsrKeySize 2048
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn sslresetcfg
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn clrsel
 
REM # Set to high performance profile (reported on proxmox forums as having a impact on performance)
%racadmPath%racadm.exe  -r %idracTarget% -u root -p calvin --nocertwarn set BIOS.SysProfileSettings.SysProfile PerfOptimized
%racadmPath%racadm.exe  -r %idracTarget% -u root -p calvin --nocertwarn jobqueue create BIOS.Setup.1-1
 
REM # Reboot iDRAC
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn racreset
ping %idractarget% -t
 
REM # Create our standard account
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Tuning.DefaultCredentialWarning Disabled
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Users.3.UserName $IDRAC_USER
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Users.3.Password $IDRAC_PASSWORD
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Users.3.Privilege 0x1ff
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Users.3.Enable 1
 
REM # Disable the default user account
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn set iDRAC.Users.2.Enable 0
  
REM # Optional if you lose your license (note: you may need to remove any lines with white spaces as the cli export seems to add them and you get a error saying the file is corrupt)
%racadmPath%racadm.exe -r %idracTarget% -u root -p calvin --nocertwarn license import -f 21S0GB2_FD00000005765970.xml -c iDRAC.Embedded.1
```



## Mellanox 

Below is a quick reference for upgrading the firmware of Mellanox Connect-X series cards. 

### Prerequisites

- A computer with Ubuntu/Debian installed and a internet connection. 
- Download the NVIDIA Firmware Tools (MFT) tools for Debian here: http://www.mellanox.com/page/management_tools. Direct link for Debian x64 MST tools v4.2.0.99 is [here](https://www.mellanox.com/downloads/MFT/mft-4.21.0-99-x86_64-deb.tgz)


Installation:
```shell
sudo apt-get install -y gcc make dkms linux-headers-`uname -r`
sudo dpkg -i mft_4.21.0-99_amd64.deb
```


Start Mellanox MST:
```shell
root@debian:~/mft-4.11.0-103-x86_64-deb# mst start

Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
Loading MST PCI configuration module - Success
Create devices
```

Confirm the card is present:
```
root@debian:~/Downloads/mft-4.11.0-103-x86_64-deb# mst status -v
MST modules:
------------
    MST PCI module loaded
    MST PCI configuration module loaded
PCI devices:
------------
DEVICE_TYPE             MST                           PCI       RDMA            NET                       NUMA 
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pciconf3     
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pci_cr3       06:00.0                   net-enp6s0d1,net-enp6s0   -1   
 
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pciconf2     
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pci_cr2       04:00.0                   net-enp4s0,net-enp4s0d1   -1   
 
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pciconf1     
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pci_cr1       03:00.0                   net-enp3s0,net-enp3s0d1   -1   
 
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pciconf0     
ConnectX3Pro(rev:0)     /dev/mst/mt4103_pci_cr0       01:00.0                   net-enp1s0d1,net-enp1s0   -1   
```

Upgrade the Firmware:
```
root@debian:~# mlxfwmanager --online -u

Querying Mellanox devices firmware ...
 
Device #1:
----------
 
  Device Type:      ConnectX3Pro
  Part Number:      MCX314A-BCC_Ax
  Description:      ConnectX-3 Pro EN network interface card; 40GigE; dual-port QSFP; PCIe3.0 x8 8GT/s; RoHS R6
  PSID:             MT_1090111023
  PCI Device Name:  /dev/mst/mt4103_pci_cr3
  Port1 MAC:        ec0d9a122660
  Port2 MAC:        ec0d9a122661
  Versions:         Current        Available    
     FW             2.40.5030      2.42.5000    
     PXE            3.4.0746       3.4.0752     
 
  Status:           Update required
 
Device #2:
----------
 
  Device Type:      ConnectX3Pro
  Part Number:      MCX314A-BCC_Ax
  Description:      ConnectX-3 Pro EN network interface card; 40GigE; dual-port QSFP; PCIe3.0 x8 8GT/s; RoHS R6
  PSID:             MT_1090111023
  PCI Device Name:  /dev/mst/mt4103_pci_cr2
  Port1 MAC:        7cfe90ac0260
  Port2 MAC:        7cfe90ac0261
  Versions:         Current        Available    
     FW             2.42.5000      2.42.5000    
     PXE            3.4.0752       3.4.0752     
 
  Status:           Up to date
 
Device #3:
----------
 
  Device Type:      ConnectX3Pro
  Part Number:      MCX314A-BCC_Ax
  Description:      ConnectX-3 Pro EN network interface card; 40GigE; dual-port QSFP; PCIe3.0 x8 8GT/s; RoHS R6
  PSID:             MT_1090111023
  PCI Device Name:  /dev/mst/mt4103_pci_cr1
  Port1 MAC:        ec0d9a1190a0
  Port2 MAC:        ec0d9a1190a1
  Versions:         Current        Available    
     FW             2.40.5030      2.42.5000    
     PXE            3.4.0746       3.4.0752     
 
  Status:           Update required

Device #4:
----------
 
  Device Type:      ConnectX3Pro
  Part Number:      MCX314A-BCC_Ax
  Description:      ConnectX-3 Pro EN network interface card; 40GigE; dual-port QSFP; PCIe3.0 x8 8GT/s; RoHS R6
  PSID:             MT_1090111023
  PCI Device Name:  /dev/mst/mt4103_pci_cr0
  Port1 MAC:        e41d2d24c610
  Port2 MAC:        e41d2d24c611
  Versions:         Current        Available    
     FW             2.42.5000      2.42.5000    
     PXE            3.4.0752       3.4.0752     
 
  Status:           Up to date
 
---------
Found 2 device(s) requiring firmware update...
 
Perform FW update? [y/N]: y
 
Please wait while downloading MFA(s) 100%
Device #1: Updating FW ...    
Done
Device #2: Up to date
Device #3: Updating FW ...    
Done
Device #4: Up to date
 
Restart needed for updates to take effect.
```

Reset to default settings on all Mellanox cards:
```shell
root@debian:~# mlxconfig reset
 
 Reset configuration for all devices? (y/n) [n] : y
Applying... Done!
-I- Please reb oot machine to load new configurations.


```

If you are not using the cards for booting, or PXE then its possible to disable it to help speed up the boot process. 

```shell
root@debian:~# mlxconfig -d /dev/mst/mt4103_pciconf0 set BOOT_OPTION_ROM_EN_P1=0 BOOT_OPTION_ROM_EN_P2=0 LEGACY_BOOT_PROTOCOL_P1=0 LEGACY_BOOT_PROTOCOL_P2=0
 
Device #1:
----------
 
Device type:    ConnectX3Pro   
Device:         /dev/mst/mt4103_pciconf0
 
Configurations:                              Next Boot       New
         BOOT_OPTION_ROM_EN_P1               True(1)         False(0)       
         BOOT_OPTION_ROM_EN_P2               True(1)         False(0)       
         LEGACY_BOOT_PROTOCOL_P1             PXE(1)          None(0)        
         LEGACY_BOOT_PROTOCOL_P2             PXE(1)          None(0)
 
 Apply new Configuration? (y/n) [n] : y
Applying... Done!
-I- Please reboot machine to load new configurations.
```

Delete boot rom (optional)

Although we have disabled pxe boot it still doesn't prevent the boot rom from loading. As the mellanox cards are not using PXE boot we can simply go ahead and remove the entire boot rom.
The process must be repeated for each card installed in the system

```shell
root@debian:~# flint -d /dev/mst/mt4103_pciconf0 q
root@debian:~# flint -d /dev/mst/mt4103_pciconf0 -allow_rom_change drom
 
-I- Preparing to remove ROM ...
Removing ROM image    - OK 
Restoring signature  - OK
```


## Intel X710 FIrmware 

Download the latest"Non-Volatile Memory (NVM) Update Utility for Intel® Ethernet Network Adapter 700 Series" pack. The link posted below will probably have expired as Intel has a habit of changing around the URLs. 

```shell
wget https://downloadmirror.intel.com/739636/700Series_NVMUpdatePackage_v9_00.zip
unzip 700Series_NVMUpdatePackage_v9_00.zip -d /tmp/
rm 700Series_NVMUpdatePackage_v9_00.zip
 
cd /tmp/700Series/Linux_x64
./nvmupdate64e
 
 
Intel(R) Ethernet NVM Update Tool
NVMUpdate version 1.37.1.1
Copyright(C) 2013 - 2021 Intel Corporation.
 
 
WARNING: To avoid damage to your device, do not stop the update or reboot or power off the system during this update.
Inventory in progress. Please wait [***|......]
 
 
Num Description                          Ver.(hex)  DevId S:B    Status
=== ================================== ============ ===== ====== ==============
01) Intel(R) I350 Gigabit Network       1.99(1.63)   1521 00:002 Update not
    Connection                                                   available
02) Intel(R) Ethernet Server Adapter    6.01(6.01)   1572 00:026 Update
    OCP X710-2                                                   available
 
Options: Adapter Index List (comma-separated), [A]ll, e[X]it
Enter selection: 02
Would you like to back up the NVM images? [Y]es/[N]o: n
Update in progress. This operation may take several minutes.
[***|......]
 
# Enter the number of the adapter that states "Update available"
```
