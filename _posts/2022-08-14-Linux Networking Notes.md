---
layout: post
title: "Linux Networking Notes"
date: 2022-08-12 10:00:00 +0900
categories: [linux, networking]
tags: [networking, linux, lpic, notes]
comments: true
---


# Linux Networking Notes

A collection of rough notes for working with the networking stack in Linux


## 1. Networking

- Summary of interfaces, their MAC, and the link status `ip -br -c link show`
- A good summary of IP address associated to interfaces `ip -br -c addr show`
- List network adapters: 

 ```
 lspci | egrep -i --color 'network|ethernet'
 
 # Below requires addational packages in Debian
 hwinfo --network --short
 lshw -class network
 ```
 
-  Use ethtool to verify physical connectivity  `ethtool enp0s31f6`

## 2. Routing

- Show routing table `ip -c route`

### 2.1. Non presistent routes 

- Route via external gateway: `ip route add {NETWORK/MASK} via {GATEWAYIP}`
- Route via an interface: `ip route add {NETWORK/MASK} dev {DEVICE}`
- Default route via a interface: `ip route add default {NETWORK/MASK} dev {DEVICE}`
- Default route via a external gateway: `ip route add default {NETWORK/MASK} via {GATEWAYIP}`

### 2.2. Permement routes
```shell
# /etc/network/interfaces
up /sbin/ip route add {NETWORK/MASK} via {GATEWAYIP} dev eth0
down /sbin/ip route delete {NETWORK/MASK} via {GATEWAYIP} dev eth0
```

## 3. Link Layer Discovery Protocol (LLDPD)

- Install lldpd `sudo apt install lldpd`
- Show LLDP Neighbours (e.g. upstream switch) `lldpcli show neighbors`


## 4. Bonding




### 4.1. Linux Bond Link Aggregation Modes

| Mode | Policy | Fault Tolerent | Load Balancing | Description
| :- | :- | :- | :- | :- 
| 0 | Round Robin | No | Yes | packets are sequentially transmitted/received through each interfaces one by one.
| 1 | Active Backup | Yes | No  | one NIC active while another NIC is asleep. If the active NIC goes down, another NIC becomes active. only supported in x86 environments.	
| 2| XOR (Exclusive OR) | Yes | Yes | In this mode the, the MAC address of the slave NIC is matched up against the incoming request’s MAC and once this connection is established same NIC is used to transmit/receive for the destination MAC.	
| 3 | Broadcast | Yes | No | 	All transmissions are sent on all slaves
| **4** | Dynamic Link Aggregation | Yes | Yes | Switch needs to support  IEEE 802.3ad (e.g. LACP)
| 5 | Transmit Load Balancing (TLB) | Yes | Yes | The outgoing traffic is distributed depending on the current load on each slave interface. Incoming traffic is received by the current slave. If the receiving slave fails, another slave takes over the MAC address of the failed slave.
| 6 | Adaptive Load Balancing (ALB) | Yes | Yes | Unlike Dynamic Link Aggregation, Adaptive Load Balancing does not require any particular switch configuration. Adaptive Load Balancing is only supported in x86 environments. The receiving packets are load balanced through ARP negotiation.

*Source: https://www.mybluelinux.com/bonding-teaming-802.3ad-lacp-on-debian-11-bullseye/#linux-bond-link-aggregation-modes*

**Miimon**
Miimon is one of the options available for monitoring the status of bond links with the other option being the usage of arp requests. This guide will use miimon. bond-miimon 100 tells the kernel to inspect the link every 100 ms. bond-downdelay 200 means that the system will wait 200 ms before concluding that the currently active interface is indeed down. The bond-updelay 400 is used to tell the system to wait on using the new active interface until 400 ms after the link is brought up. most importantly, updelay and downdelay, both of these values must be multiples of the miimon value otherwise the system will round down.


-  Verify bond status `cat /proc/net/bonding/bond0`

### 4.2. Configure a host for bonding with lacp:

```shell
 # Bonding Support (Old way, see below for ifupdown2)
 sudo apt install ifenslave
 modprobe bonding
 echo 'bonding' >> /etc/modules
 
 # VLAN Support 
 apt install vlan
 modprobe 8021q
 echo '8021q' >> /etc/modules 
```


### 4.3. Bonds with VLAN interfaces 

Example of the network interface files setup to use LACP with Jumbo frames:
```
# /etc/network/interfaces

auto bond2
iface bond2 inet manual
	bond-slaves ens3 ens3d1
	bond-miimon 100
	bond-mode 802.3ad
	bond-xmit-hash-policy layer2+3
	mtu 9000
	bond-lacp-rate 1
```

Example of the network interfaces file using a bridge interface connecting to a bond with a vlan interface:
```
# /etc/network/interfaces

auto vmbr2
iface vmbr2 inet static
	address 172.17.230.43/27
	bridge-ports bond2.231
	bridge-stp off
	bridge-fd 0
	mtu 9000
```

## 5. ifupdown2

The `ifupdown2` is a package developement by Cumulus Networks allowing for easier operations whenw working with the `/etc/network/interfaces` files. Instead of working with the `ifup` and `ifdown` commands or restart a PC for changes to take effect. 

````
apt -y install ifupdown2
````
>This installation may reset your network connections.
{: .prompt-warning }

- ifupdown2 can be used to reload all the network interfaces without having to restart the PC `ifreload -all`

## 6. L2TP over IPSEC VPN Setup CLI

- `sudo apt install strongswan xl2tpd`
- `strongswan-starter`
- `ipsec up myvpn`

