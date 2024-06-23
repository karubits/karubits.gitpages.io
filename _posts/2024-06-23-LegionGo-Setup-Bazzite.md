---
layout: post
title: "Legion Go Bazzite Setup (Post Installation)"
date: 2023-06-23 01:00:00 +0900
categories: [gaming]
tags: [gaming, legiongo, bazzite, linux]
---
## Legion GO Bazzite Setup (Post Installation)
- After installation on the setup wizzard select Decky Loader for installation. Reboot the device and go through the Bazzite/Deck setup wizard. 
- Press on the Right-hand side settings button to bring up the side menu -> Then go to Plug icon -> Market Place icon -> Sort by most downloaded -> Select CSS Loader and install.  
- After the device setup has been completed, press the Legion button to go back and exit to desktop. 
- Type in `ujust setup-decky` (it should say "Decky Loader" is installed) and install SimpleDeckyTDP
- Run the following installers
   ```
   ujust install-hdd-controller-glyph-theme
   ujust install-legion-go-theme

   # Reboot
   sudo reboot now
   ```
- Once restarted. Press on the Right-hand side settings button to bring up the side menu -> Then go to Plug icon -> CSS Loader
- Enable "Handheld Controller Glpyhs" -> Select Legion Go in the Handheld drop box. 
- Enable "Legion Go Theme"


### Optional and personalized steps
- Run the emudeck installer if you want to use emuation on the desktop. 
- Run tailscale (Optional). Tailscale is already installed by default with Bazzite. `tailscale up`
- SSH setup
   ```
   # enable ssh
   systemctl enable sshd
   systemctl start sshd

   # Setup ssh keys
   mkdir ~/.ssh
   curl https://github.com/{{ github username }}.keys >> ~/.ssh/authorized_keys

   # Disable password authentication
   sudo sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
   sudo systemctl restart sshd
   ```

