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

Factors      	       	                         | Description                                                                                      | Command
------------------------------------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------
No PIN or touch are required                     | You will not be required to enter your FIDO2 PIN or touch your YubiKey each time to authenticate | `ssh-keygen -t ed25519-sk -O resident -O no-touch-required`
PIN but no touch required                        | Entering the PIN will be required but touching the physical key will not                         | `ssh-keygen -t ed25519-sk -O resident -O verify-required -O no-touch-required`
No PIN but touch is required                     | You will only need to touch the YubiKey to authenticate                                          | `ssh-keygen -t ed25519-sk -O resident`
A PIN and a touch are required (**most secure**) | This is the most secure option, it requires both the PIN and touching to be used                 | `ssh-keygen -t ed25519-sk -O resident -O verify-required`


ref. 
- [Securing SSH with FIDO2](https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html)
- [GitHub now supports SSH security keys](https://www.yubico.com/blog/github-now-supports-ssh-security-keys/)


Using a Discoveral (resident) key has several benefits:

1. Portability: Since the private key is stored on the YubiKey, you can use it on multiple devices without having to copy or transfer the key between them. Just plug in the YubiKey, and you can use the resident key for authentication.
2. Enhanced security: Resident keys are protected by the hardware security of the YubiKey. This makes it more difficult for an attacker to access or compromise the private key compared to when it is stored on a computer's filesystem.

However for the best security a non-discoverable credential is preferred. If the key was found they would not be able to use it for ssh access.