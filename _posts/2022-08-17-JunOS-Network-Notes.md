---
layout: post
title: "Juniper JunOS Networking Notes"
date: 2022-08-14 10:00:00 +0900
categories: [network, junos]
tags: [network, junos, switches]
---

# Juniper EX3300

## Native VLANs on EX3300

Unlike ELS with the EX4300 the configuration is different for EX330. This has caught me off guard a few times. 

```
set interfaces interface-range SERVER-A member ge-0/0/6
set interfaces interface-range SERVER-A unit 0 family ethernet-switching port-mode trunk
set interfaces interface-range SERVER-A unit 0 family ethernet-switching vlan members V12_SHARED_SERVICES
set interfaces interface-range SERVER-A unit 0 family ethernet-switching native-vlan-id 13
```
Notice how the native vlan should not be a member of the vlans unlike EX4300s. 

> EX3300 will accept the EX4300 commands and apply it to the configuration however it doesn't mean its supported. Use the show configuration command without display set to see if the command is actually supported. 
   ```
           unit 0 {
            family ethernet-switching {
                ##
                ## Warning: statement ignored: unsupported platform (ex3300-48p)
                ##
                interface-mode access;
                vlan {
                    members V13_INBAND_MGMT;

   ```
{: .prompt-tip }

# Juniper ELS Examples (based on EX4300)

## Native VLANs on EX4300 ELS Switches
```
# Create VLANS
set vlans V1001_EMPLOYEE description "[1001] Employee Network (10.8.240.1/24)"
set vlans V1001_EMPLOYEE vlan-id 1001
set vlans V1002_BYOD description "[1002] Employee Network (10.8.241.1/24)"
set vlans V1002_BYOD vlan-id 1002
# ....etc


# Configure trunk with native VLANS
set interfaces interface-range UNIFI_AP member ge-0/0/47
set interfaces interface-range UNIFI_AP member ge-1/0/47
set interfaces interface-range UNIFI_AP native-vlan-id 129
set interfaces interface-range UNIFI_AP unit 0 family ethernet-switching interface-mode trunk
set interfaces interface-range UNIFI_AP unit 0 family ethernet-switching vlan members EMPLOYEE
set interfaces interface-range UNIFI_AP unit 0 family ethernet-switching vlan members BYOD
set interfaces interface-range UNIFI_AP unit 0 family ethernet-switching vlan members GUESTS
set interfaces interface-range UNIFI_AP unit 0 family ethernet-switching vlan members MANAGEMENT
set poe interface UNIFI_AP
```

> Even when setting the native-vlad-id the vlan must also exist as a member of the trunk. 
{: .prompt-tip }

## Configure LACP uplink

1. Example below
   ```
   set interfaces ae1 description "Uplink to Server SRV001"
   set interfaces ae1 aggregated-ether-options lacp active
   set interfaces ae1 aggregated-ether-options lacp periodic fast
   set interfaces ae1 unit 0 family ethernet-switching interface-mode access
   set interfaces ae1 unit 0 family ethernet-switching vlan members MANAGEMENT
   
   set interfaces interface-range SRV001 member ge-0/0/0
   set interfaces interface-range SRV001 member ge-1/0/0
   set interfaces interface-range SRV001 ether-options 802.3ad ae1
   ```


2. Don't forget to bump the aggregate count for each aggregate created
   ```
   set chassis aggregated-devices ethernet device-count 32
   ```

## Upgrading firmware

Assumption is a http server is available to serve the firmware images. 

1. On some occasions space has to be made before upgrading the firmware. 
   ```
   # Single Switch (e.g. EX3300s)
   file delete-directory /var/tmp/.schema-cache recurse
   request system storage cleanup all-members | no-more
    
   #  switches in a virtual chassis
   file delete-directory fpc0:/var/tmp/.schema-cache recurse
   file delete-directory fpc1:/var/tmp/.schema-cache recurse
   file delete-directory fpc2:/var/tmp/.schema-cache recurse
   file delete-directory fpc3:/var/tmp/.schema-cache recurse
   file delete-directory fpc4:/var/tmp/.schema-cache recurse
   file delete-directory fpc5:/var/tmp/.schema-cache recurse
   file delete-directory fpc6:/var/tmp/.schema-cache recurse
   request system storage cleanup all-members no-confirm | no-more
   ```

2. Upgrade the firmware
   ```
   request system software add http://[ IP ADRDRESS ]:[PORT]/[PATH]
   e.g.
   request system software add http://10.21.24.50:8000/junos/ex4300/jinstall-ex-4300-21.4R3-S3.4-signed.tgz
   ```
3. Reboot the switch
   ```
   # Instant reboot
   request system reboot message "Rebooting to upgrade to 15.1R7-S13" 
   
   # Instant reboot for virtual chassis
   request system reboot all-members message "Rebooting to upgrade to 21.4R3-S3.4"
   
   # Scheduled reboot 
   request system reboot at "2023-05-05 23:00:00" message "Rebooting to upgrade to 15.1R7-S13" 
   
   # Scheduled reboot for virtual chassis
   request system reboot at "2023-05-05 23:00:00" all-members message "Rebooting to upgrade to 21.4R3-S3.4"
   ```

## Upgrading firmware from USB

1. Log into the switch and drop out of the cli to the shell, then mount the USB device and copy the junos image over to a local tmp folder. 
   ```bash
   start shell
   mount_msdosfs /dev/da1s1 /mnt
   cp /mnt/jinstall-ex-3300-15.1R7-S13-domestic-signed.tgz /var/tmp/
   cli
   ```
2. Upgrade 
   ```
   request system software add /var/tmp/jinstall-ex-3300-15.1R7-S13-domestic-signed.tgz 
   ```


> If you see the error "certificate is not yet valid: /C=US/ST=CA/L=Sunnyvale/O=J...." manually set the date by using `set date 202307101811.12`
{: .prompt-tip }

## Factory Wipe and install new firmware from USB

- To perform this recovery installation, the USB device should be formatted to FAT-32 and should be empty (recommended USB size: 1GB, 2GB, or 4GB). Review the complete USB compatibility specifications listed in USB Port Specifications for an EX Series Switch.
- Copy the Junos OS package to the USB device.
- Power off the EX switch.
- Plug the USB device into the EX switch.
- Power on the EX switch.
- When you see the `Hit [Enter] to boot immediately, or space bar for command prompt` message prompt appear, press the Space bar to get the loader prompt.


> To avoid missing it, you may start pressing the Space bar some seconds before the message prompt appears.
{: .prompt-tip }


- Issue the `install` command with the `format` option:
```bash
loader> install --format file:///<Junos package name>
# For example:
loader> install --format file:///jinstall-ex-4200-15.1R7-S6.3-domestic-signed.tg
```


> Note the above does require 3x forward slashes after file:, `///`
{: .prompt-tip }

# Other Notes

Ensure the LLDP responder shows the interface name instead of the interface index
```
set protocols lldp port-id-subtype interface-name
```

Configure a management IP Address:
```
# Create a irb interface
interfaces irb unit 1 family inet address 10.8.251.41/24

# Map the irb to the VLAN
set vlans INBAND-MANAGEMENT l3-interface irb.1

# Create a default route
set routing-options static route 0.0.0.0/0 next-hop 10.8.251.1
```

Somtimes with the EX4300 the text can be cut off if your hostname is too long on the lcd screen and it can be hard to identify which switch is which. You can set a custom display messege on the LCD instead. 
``` 
set chassis display fpc-slot 5 message BROKEN-SWITCH
``` 
