---
layout: post
title: "Windows Server DHCP and PowerShell"
date: 2022-12-27 10:00:00 +0900
categories: [windows server]
tags: [dhcp, windows, powershell]
comments: true
---
# Windows Server DHCP and PowerShell

I was recently tasked with setting Windows DHCP Server with powershell. Coming from isc-dhcp-server on Linux some basic options such as DNS Search Suffix (113) or setting up scopes in bulk can be tedious through the GUI, so this post details some common tasks that one might go through for setting up DHCP server from scratch. 


### Scopes
