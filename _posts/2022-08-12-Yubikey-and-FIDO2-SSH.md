---
layout: post
title: "Yubikey and FIDO 2 with SSH"
date: 2022-08-14 10:00:00 +0900
categories: [linux, ssh]
tags: [yubikey, linux, ssh, fido]
---


## Initial Setup

To start using FIDO on you Yubikey you will need to set a PIN first. This can be done from the command line using the Yubikey manager application. 

```bash
# If you don't have ykman in
sudo apt install yubikey-manager -y

# Check if a PIN has been set:
ykman fido info

# Set the PIN:
ykman fido access change-pin
````
> Resetting the PIN will remove all FIDO2 credentials. 
{: .prompt-danger }

> The FIDO2 security key will be locked if the PIN is incorrect 8 times in a row. In that case, you will need to reset the security key. After resetting, you will lose all the authentication information you have registered so far.
{: .prompt-danger }


## Key Types

With SSH and FIDO you will have two primary options for generating a ssh key. </br>

1. **Resident Key** (Discoverable) - Allows for portability. Requires OpenSSH 8.3 or above. 
2. **Non-resident Key** (Non-Discoverable) - Requires a credential id file that is automatically generated in the `~/.ssh`.Requires OpenSSH 8.2p1 or above. Recommended for high security environments.

### Generating a Resident Key (Discoverable)

Before generating your ssh key its important to understand the various options you have for resident keys. 

| Factors | Command |Description | 
| :--- | :--- | :--- | 
| No PIN or touch are required|  `ssh-keygen -t ed25519-sk -O resident -O no-touch-required` | You will not be required to enter your FIDO2 PIN or touch your YubiKey each time to authenticate |
| PIN but no touch required | `ssh-keygen -t ed25519-sk -O resident -O verify-required -O no-touch-required` | Entering the PIN will be required but touching the physical key will not | 
| No PIN but touch is required| `ssh-keygen -t ed25519-sk -O resident` | You will only need to touch the YubiKey to authenticate | 
| A PIN and a touch are required (**most secure**) | `ssh-keygen -t ed25519-sk -O resident -O verify-required` | This is the most secure option, it requires both the PIN and touching to be used |


To create a new SSH key on your Yubikey, here are some examples below:

```bash
# Example
ssh-keygen -t ed25519-sk -O resident \
    -O application=ssh:your-text-here \
    -C "comment or identifier"


# Actual Example
ssh-keygen -t ed25519-sk -O resident \
    -O application=ssh:ssh_key_for_server_access \
    -C "Emergency Key 1 - 10532500" 

# Actual example with PIN prompting
ssh-keygen -t ed25519-sk -O resident -O verify-required \
    -O application=ssh:ssh_key_for_server_access \
    -C "Emergency Key 2 - 10653212"

# Verify the key
❯ ykman fido credentials list
Enter your PIN: 
ssh:ssh_for_servers_20231230 0000000000000000000000000000000000000000000000000000000000000000 openssh
````

When using PIN you will need to ensure ssh-askpass is installed and the ssh-agent is loaded and the key is loaded using the -K paramter with the ssh-add option. 

```bash
# Install ssh-askpass
sudo apt install ssh-askpass -y

# Sometimes it is required to set the SSH_ASKPASS env variable for the binary location 
SSH_ASKPASS=$(which ssh-askpass)

# Start the SSH Agent
eval "$(ssh-agent -s)" 

# Load the resident fido key
ssh-add -K

# Verify the key is loaded
ssh-add -L
```

Alternatively setup a shortcut in your bashrc to make it easier. 

```bash
export SSH_ASKPASS=/usr/bin/ssh-askpass
alias yubikey_ssh='eval "$(ssh-agent -s)" && ssh-add -K && ssh-add -L'
```

> When connecting you will have a GUI prompt to enter your PIN number. After entering your PIN the command prompt will look like its handing, simply touch the Yubikey to complete the connection. 
{: .prompt-info }

If you would like to delete the ssk key from your Yubikey you can use the following command: </br>
`ykman fido credentials delete ssh`

## References

- [Securing SSH with FIDO2](https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html)
- [GitHub now supports SSH security keys](https://www.yubico.com/blog/github-now-supports-ssh-security-keys/)