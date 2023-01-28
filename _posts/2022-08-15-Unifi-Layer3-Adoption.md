---
layout: post
title: "Unifi Layer 3 Adoption"
date: 2022-08-14 10:00:00 +0900
categories: [network, unifi]
tags: [network, unifi, network, controller, ubiquiti]
comments: true
---

# Adoption Methods

There are three options for adopting devices over different networks. 
1. DNS
2. Using DHCP options
3. Manual configuration

## DNS
----

If you have a internal DNS server that you manage you can simply add a dns A record for `unifi` pointing to the IP address of your unif controller. As soon as Unifi devices it will begin trying to resolve `unifi` and if successful it will connect to the Unifi Network Controller. 

## DHCP Adoption
----

Unifi uses option 43 vendor-specifc to specify the Unifi Controller IP address. 

### pfsense

Pfsense ues hex code format to specify the unifi controller ip address. To concert an IP address to hex you can use the `gethostip` command from the syslinux-utils packge in Debain/Ubuntu. 

1. Convert the IP to hex
```shell
sudo apt install -y syslinux-utils
gethostip -x 192.168.46.250
C0A82EFA
````
2. Then append the suboption of `01` and payload lengh of `04` in front of the hex output. The final result would be be `01:04:C0:A8:2E:FA`
3. Log into pfsense ➡️ Services ➡️ DHCP Server and scroll down to the bottom of the page to "Other Options" ➡️ Addational BOOTP/DHCP Options. And add the option 43 hex string as pictured below:
![Pfsense DHCP Option](/img/pfsense-dhcp-option.png)
_PFsense DHCP Option for Unifi L3 Adoption_


### isc-dhcp-server

1. In the `/etc/dhcp/dhcpd.conf` you will first need to define vendor dhcp option for Unifi.
```shell
option space ubnt;
option ubnt.unifi-address code 1 = ip-address;
class "ubnt" {
        match if substring (option vendor-class-identifier, 0, 4) = "ubnt";
        option vendor-class-identifier "ubnt";
        vendor-option-space ubnt;
}
``` 
2. Then in your scope you can simplly put the IP address of the Unifi Network Controller.  
```shell
subnet 192.168.50.0 netmask 255.255.255.0 {
        option domain-name-servers 1.1.1.1, 8.8.8.8;
        option routers 192.168.50.1;
        option ubnt.unifi-address 192.168.46.250;  ##<----- Unifi Controller IP Address
        pool {
            range 192.168.50.50 192.168.50.250;
        }
}
```
3. Be sure to restart the dhcp server for the changes to take effect. `systemctl restart isc-dhcp-server`

### Windows Server DHCP

1. On the DHCP Server open a Powershell Terminal (As administration). 
   ```shell
   $UNIFI_CONTROLLER_IP = "172.16.1.4"
   $SCOPE = "192.168.4.0" 
   
   # Create the vendor class
   Add-DhcpServerv4Class -Name Ubiquiti -data    "ubnt" -Type "Vendor"
   
   # Create a new DHCP option under the vendor    class 
   Add-DhcpServerv4OptionDefinition -Name "UniFi    Controller" -OptionId 1 -Type "BinaryData"    -VendorClass "Ubiquiti" -Description "Unifi    Controller IP as Hex Object"
   
   # Convert the controller IP into command    separated hex in the expected format.
   $ip = "$UNIFI_CONTROLLER_IP"
   $octets = $ip.Split(".")
   $hexOctets = @()
   foreach ($octet in $octets) {
       $hexOctets += "0x{0:X2}" -f [int]$octet
   }
   $hexOctets -join ", "
   
   # Add the controller IP option to the targeted    scope
   Get-DhcpServerv4Scope $SCOPE |    Set-DhcpServerv4OptionValue -VendorClass    'Ubiquiti' -OptionId 001 -Value $hexOctets
   ```



## Manual Adoption (SSH)
----


1. Connect the Unifi device to the network
2. Locate the IP address of the device and SSH into the device. The default user is `ubnt` and password is `ubnt`. 
3. Then set the inform URL to the hostname or IP address of your Unifi Network Controller. 
```shell
# Example of setting the inform URL with a FQDN
set-inform http://unifi.karubuts.com:8080/inform
# Example of setting the inform URL with an IP address. 
set-inform http://192.168.46.250:8080/inform
```
4. To confirm the inform URL has been set simple type `info` in the command prompt. 
```shell
USW-Pro-48-PoE.6.2.14# info
Model:       USW-Pro-48-PoE
-
Version:     6.2.14.13855
MAC Address: 24:7a:4c:89:6a:1b
IP Address:  192.168.20.10
Hostname:    USW-Pro-48-PoE
Uptime:      6509224 seconds
-
Status:      Connected (http://unifi.karubuts.com:8080/inform)
```
5. The device should be now visable in the Unifi Network Controller dashboard. 


# References
- _[UniFi Network - Layer 3 Adoption](https://help.ui.com/hc/en-us/articles/204909754-UniFi-Layer-3-Adoption-for-Remote-UniFi-Network-Applications)_
- _[TCPIP.WTF - Unifi L3 Adoption with DHCP Option 43 on pfSense, Mikrotik and others](https://tcpip.wtf/en/unifi-l3-adoption-with-dhcp-option-43-on-pfsense-mikrotik-and-others.htm)_
