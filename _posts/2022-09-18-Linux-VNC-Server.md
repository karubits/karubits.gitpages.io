---
layout: post
title: "Linux VNC Server"
date: 2022-09-17 10:00:00 +0900
categories: [linux]
tags: [linux, vnc]
comments: true
---


# How to setup a VNC Server on Linux

### On the PC / Desktop you want to access

1. Install the X11 Server
```shell
sudo apt install -y x11vnc
```
2. Then setup the initial password
```shell
x11vnc -usepw
```
Press Ctrl+C to close the vnc server session. 

### On the remote PC

1.  You can add the command to start the vnc server on the remote PC in your `~/.ssh/config` as below. This approach means the vnc-server is only running when you need it. 
```shell
Host remote-pc-vnc
        User karubits
        Hostname 192.168.139.128
        LocalForward 5901 localhost:5901
        RemoteCommand x11vnc -localhost -display :1 -usepw -rfbport 5901 -ncache 10
```
Then ssh the remote PC you want to see the desktop on
```shell
ssh remote-pc-vnc
```
Then open  your vnc viewer and connect to the localhost of your PC. 

![VNC Viewier - Localhost](/img/vnc-viewer.png)
_VNC Viewer - localhost connection_


