---
layout: post
title: "Juniper JunOS Networking Notes"
date: 2022-08-14 10:00:00 +0900
categories: [network, junos]
tags: [network, junos, switches]
---


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


Don't forget to bumb the agregate count for each aggregate created
```
set chassis aggregated-devices ethernet device-count 32
```


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
