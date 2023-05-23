---
layout: post
title: "Setup Debian/Ubuntu (Gnome) with Passwordless Logins (U2F and Fingerprint)"
date: 2023-04-8 03:00:00 +0900
categories: [linux, server]
tags: [yubikey, linux, debian, u2f, fingerprint, passwordless]
---

Setup Debian/Ubuntu (Gnome) with Passwordless Logins (U2F and Fingerprint)

A short guide covering U2F (With a Yubikey) and Fingerprint logins for your PC.

1. Instal the U2F Pam Module
   ```bash
   sudo apt install -y libpam-u2f yubikey-manager yubikey-manager-qt
   ```
2. Set a FIDO PIN on your Yubikey (If you haven't done so already)
   ```
   ykman fido access change-pin
   ```
3. Create a user to u2f mapping while your Yubikey is inserted. You can add as many keys as you like. Also it is good idea to set the permission of the newly created file to 600.
   ```
   pamu2fcfg -u$(whoami) | sudo tee /etc/u2f_mappings
   sudo chmod 600 /etc/u2f_mappings
   ```
4. Edit the gdm-password file.
   ```bash
   sudo nano /etc/pam.d/gdm-password
   ```
5. Then add the following line at the top of the auth section of the gmd-password file.
   ```bash
   auth    sufficient      pam_u2f.so authfile=/etc/u2f_mappings cue pinverification=1
   ```
- The pinverification=1 will ensure the you are prompted for a PIN. The will prevent anyone with access to your Yubikey to login into your PC unless they know the PIN of your Yubikey.

ref. https://developers.yubico.com/pam-u2f/


Fingerprint Setup and Enrolement

In addition to U2F we can also enroll a finger on your laptop.

1. Install the required pacakges. 
   ```bash
   sudo apt install fprintd libpam-fprintd
   ```
2. Enable the pam module. 
   ```bash
   sudo pam-auth-update --enable fprintd
   ```
3. Enroll your finder. You can choose the following parameters for the finger type. left-thumb, left-index-finger, left-middle-finger, left-ring-finger, left-little-finger, right-thumb, right-index-finger, right-middle-finger, right-ring-finger, right-little-finger.
   ```bash
   fprintd-enroll -f right-middle-finger
   ```

Then enrolled signatures are stored here:
/var/lib/fprint
