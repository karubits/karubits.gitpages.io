---
layout: post
title: "Yubikey and FIDO 2 with SSH"
date: 2022-08-14 10:00:00 +0900
categories: [linux, ssh]
tags: [yubikey, linux, ssh, fido]
---




To set a FIDO2 pin on a new device:
````
# Check if a PIN has been set:
ykman fido info

# Set the PIN:
ykman fido access change-pin
````
Warning: Resetting the PIN will remove all FIDO2 credentials. 


```
# Example
ssh-keygen -t ed25519-sk -O resident \
    -O application=ssh:your-text-here \
    -C "$(date +'%d-%m-%Y')-physical_yubikey_number"



# Actual execution 
ssh-keygen -t ed25519-sk -O resident \
    -O application=ssh:emergency-ssh-access-desktop-server \
    -C "Emergency Key 1 - 10532500" 


ssh-keygen -t ed25519-sk -O resident \
    -O application=ssh:emergency-ssh-access-desktop-server \
    -C "Emergency Key 2 - 10653212"
````

ref. https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html

