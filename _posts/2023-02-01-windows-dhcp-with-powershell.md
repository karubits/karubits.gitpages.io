---
layout: post
title: "Windows Server DHCP and PowerShell"
date: 2023-02-01 03:00:00 +0900
categories: [windows server]
tags: [dhcp, windows, powershell]
comments: true
---

I was recently tasked with setting Windows DHCP Server with powershell. Coming from isc-dhcp-server on Linux some basic options such as DNS Search Suffix (113) or setting up scopes in bulk can be tedious through the GUI, so this post details some common tasks that one might go through for setting up DHCP server from scratch in powershell. 

# Common DHCP Options

## DNS Search List

As the DHCP option expects an HEX array we need to convert the search list into a HEX array. 
1. Create the DHCP Option
   ```powershell
   Add-DhcpServerv4OptionDefinition -Name "DNS Search List" -OptionId 119 -Type Byte -MultiValued
   ```
2. Setup the variables for the search list and the scope you want to target. 
   ```powershell
   $DNS_SEARCH_LIST = "example1.local;coffee.local;karubits.local"
   $TARGET_SCOPE = "10.2.4.0"
   ```
3. Convert the DNS search list and add the DHCP Option to the scope. 
   ```powershell
   $splittedDomainSearchList = $DNS_SEARCH_LIST -split "\;"
   $DNS_SEARCH_LISTHexArray = @();
   
   Foreach ($domain in $splittedDomainSearchList) 
   {
       $splittedDomainParts = $domain -split "\."
       Foreach ($domainPart in $splittedDomainParts) 
       {
           $domainPartHexArray = @()
           $domainPartHexArray += $domainPart.Length
           $domainPartHexArray += $domainPart.ToCharArray();
           Foreach ($item in $domainPartHexArray) 
           {
               $DNS_SEARCH_LISTHexArray += [System.Convert]::ToUInt32($item)
           }
       }
       $DNS_SEARCH_LISTHexArray+= 0x00
   }
   
   # Add the DNS Search list as an hex array to the target scope
   Set-DhcpServerv4OptionValue -ScopeId $TARGET_SCOPE -OptionId 119 -Value $DNS_SEARCH_LISTHexArray
   ```


## TimeZone Options 
A common use case for setting the timezone options in DHCP can be for VOIP phones and getting the time to be displayed correctly.  There are two DHCP options that can be created and set. Option ID 100 and 101. 

To read more about the specifics of the option I recommend reader the following [link](https://www.lorier.net/docs/dhcp-timezone.html).

1. Create the DHCP options for 100 and 101. In this example the timezone is specified for Japan. 
   ```powershell
   Add-DhcpServerv4OptionDefinition -Name PCode    -Description "TZ-POSIX String" -OptionId 100    -Type "String" # JST-9
   Add-DhcpServerv4OptionDefinition -Name TCode    -Description "TZ-Database String" -OptionId 101    -Type "String"
   ```
2. You can set the default value globally instead of the scope level if your DHCP server resides in a single TZ. 
   ```powershell
   Set-DhcpServerv4OptionValue  -OptionId 101    -Value "Asia/Tokyo"
   Set-DhcpServerv4OptionValue  -OptionId 100    -Value "JST-9"
   ```

## Unifi Network Controller (L3 Adoption)
If you have a Unifi Network Controller you can create a vendor specific option and define the IP address of the network controller for automatic discovery of Unifi devices. 

1. First create the vendor class for Ubiquiti
   ```powershell
   Add-DhcpServerv4Class -Name Ubiquiti -data "ubnt" -Type "Vendor"
   ```
2. Create the new DHCP Option
   ```powershell
   Add-DhcpServerv4OptionDefinition -Name "UniFi Controller" -OptionId 1 -Type "BinaryData" -VendorClass "Ubiquiti" -Description "Unifi Controller IP as Hex Object"
   ```
3. We can specify the controller IP address and the DHCP scope you want to add the L3 adoption IP address to. 
   ```powershell
   $UNIFI_CONTROLLER_IP = "10.5.22.12"
   $SCOPE = "10.7.5.0" 
   ```
4. Convert the Unifi controller into a hex
   ```powershell
   $ip = "$UNIFI_CONTROLLER_IP"
   $octets = $ip.Split(".")
   $hexOctets = @()
   foreach ($octet in $octets) {
       $hexOctets += "0x{0:X2}" -f [int]$octet
   }
   $hexOctets -join ", "
   ```
5. Add the DHCP Option to the target scope. 
   ```
   # Add the controller IP option to the target scope
   Set-DhcpServerv4OptionValue -VendorClass 'Ubiquiti' -OptionId 001 -Value $hexOctets -ScopeId $SCOPE
   ```

# Example of creating DHCP scopes in Bulk

- To create DHCP scopes in bulk one approach is to define the scopes and options in a CSV file. Then use a powershell script to import the CSV and loop through each row. 
- Here is an example of a CSV. 
   ```
   Name,Description,ScopeID,StartRange,EndRange,LeaseDuration,SubnetMask,Router,DnsDomain,DNSServer,NTPServers,DNSDomainSearch
   102_EMPLOYEES,Tokyo Employees - VLAN: 102  (172.16.150.0/24),172.16.150.0,172.16.150.100,172.16.150.220,18.00:00:00,255.255.255.0,172.16.150.1,employees.karubits.com,"1.1.1.1,1.0.0.1,8.8.8.8","192.168.88.1,10.2.25.3,10.2.25.10,10.2.25.11,192.168.88.14","employees.karubits.com,global.karubits.com,karubits.local"
   103_GUEST,Tokyo Guests - VLAN: 103  (192.168.244.0/24),192.168.244.0,192.168.244.100,192.168.244.220,18.00:00:00,255.255.255.0,192.168.244.1,guests.karubits.com,"1.1.1.1,1.0.0.1,8.8.8.8","192.168.88.1,10.2.25.3,10.2.25.10,10.2.25.11,192.168.88.14",guests.karubits.com
   ```
- Then use powershell to import the CSV and loop through each row. 
   ```powershell
   # Import the CSV file
   $data = Import-Csv -Path 'dhcp_scopes.csv'
   
   # Loop through each row in the CSV file
   foreach ($row in $data) {
       # Create a new DHCP scope
       $ScopeID = $row.ScopeID
       $DnsServer = $row.DNSServer.Split(',')
       $Router = $row.Router
       $DnsDomain = $row.DnsDomain
       $NTPServers =  $row.NTPServers.Split(',')
       $DomainSearchList = $row.DNSDomainSearch
       $Name = $row.Name
   
       # Check if scope already exists
       $ScopeExists = Get-DhcpServerv4Scope | Where-Object { $_.ScopeId -eq $ScopeID }
   
       # If scope does not exist, create it
       if (!$ScopeExists) {
           Add-DhcpServerv4Scope -Name $Name -Description $row.Description -SubnetMask $row.SubnetMask -StartRange $row.StartRange -EndRange $row.EndRange -LeaseDuration $row.LeaseDuration
       } else {
           Write-Host "Scope with ID $ScopeID already exists, skipping creation"
       }
   
       # Add DNS servers, DNS Domain, and Default Gateway to the DHCP scope
       Set-DhcpServerv4OptionValue -ScopeId $ScopeID -DNSServer $DnsServer -DnsDomain $DnsDomain -Router $Router 
   
       # Add NTP servers to the DHCP scope
       Set-DHCPServerv4optionvalue -ScopeId $ScopeID -OptionId 042 -Value $NTPServers
   
       # Convert the domain search list into a HEX array
       $splittedDomainSearchList = $DomainSearchList -split "\;"
       $domainSearchListHexArray = @();
   
       # Convert the domain search list into a hex array
       Foreach ($domain in $splittedDomainSearchList) 
       {
           $splittedDomainParts = $domain -split "\."
           Foreach ($domainPart in $splittedDomainParts) 
           {
               $domainPartHexArray = @()
               $domainPartHexArray += $domainPart.Length
               $domainPartHexArray += $domainPart.ToCharArray();
               Foreach ($item in $domainPartHexArray) 
               {
                   $domainSearchListHexArray += [System.Convert]::ToUInt32($item)
               }
           }
           $domainSearchListHexArray+= 0x00
       }
   
       # Set the domain search list with its HEX value 
       Set-DhcpServerv4OptionValue -ScopeId $ScopeID -OptionId 119 -Value $domainSearchListHexArray
   
   }
   ```

# Migrating DHCP reservations from pfsense

- This python script will read the config.xml of a pfsense configuration and create a CSV file containg the MAC Address, IP Address, and description. 
   ```python
   ## Static Leases from pfsense
   
   # Python to read the config.xml and write the static leases out to csv
   import xml.etree.ElementTree as ET
   import csv
   
   # parse the xml file
   tree = ET.parse('config.xml')
   root = tree.getroot()
   
   # open a csv file for writing
   with open('static-dhcp-entries.csv', 'w', newline='') as csvfile:
       # create a csv writer
       csvwriter = csv.writer(csvfile)
       # write the header row
       csvwriter.writerow(['mac', 'ipaddr', 'descr'])
       # iterate over all staticmap elements
       for staticmap in root.findall('.//staticmap'):
           # extract the values of mac, ipaddr, and descr
           mac = staticmap.find('mac').text
           ipaddr = staticmap.find('ipaddr').text
           descr = staticmap.find('descr').text
           # write the values to the csv file
           csvwriter.writerow([mac, ipaddr, descr])
   ```
- Windows Server DHCP requires a name for the dhcp reservation and in this case they were manually added to the CSV. 
- Then to import the DHCP reservations we can use powershell to read the csv file. 
   ```powershell
   # DHCPv4 Reservations Creation Script
   # This script reads a CSV file containing DHCP reservations information and creates the reservations in the DHCP server.
   # It handles the following:
   # - CSV headers match the same name as the PowerShell parameters
   # - The scope ID uses the IP address, but changes the last octet to 0 (e.g. 192.168.2.23 becomes 192.168.2.0)
   # - The MAC address format is 00:17:fc:6f:3c:78, which is converted to 00-17-fc-6f-3c-78
   # - Assumes the -Type is set to "Both" for all reservations
   
   # Check to see if the CSV file exists. 
   $file = "pfsense-static-leases.csv"
   if (!(Test-Path $file -PathType Leaf)) {
     Write-Error "Error: File $file not found"
     return
   }
   
   # Import the DHCP reservations data from the CSV file
   Import-Csv $file | ForEach-Object {
       # Assign the values from the CSV to variables
       $ip = $_.'IP Address'
       $name = $_.'Name'
       $mac = $_.'Mac Address' -replace ':', '-'
       $description = $_.'Description'
       $scopeid = ($_.'IP Address'.Split('.')[0..2] -join '.') + '.0'
   
       # Check if the reservation already exists
       if (!(Get-DhcpServerv4Reservation -ScopeId $scopeid -ClientId $mac -ErrorAction SilentlyContinue)) {
           # If the reservation does not exist, display a message and create the reservation
           Write-Host -ForegroundColor Green "Creating reservation for $name with IP address $ip and MAC address $mac"
           Add-DhcpServerv4Reservation -ScopeId $scopeid -IPAddress $ip -ClientId $mac -Name $name -Type "Both" -Description $description
       } else {
           # If the reservation already exists, display a message and skip creating it
           Write-Host -ForegroundColor Green "Updating reservation for $name with IP address $ip and MAC address $mac as it already exists."
   	Set-DhcpServerv4Reservation -IPAddress $ip -ClientId $mac -Name $name -Type "Both" -Description $description
       }
   }
   ```
