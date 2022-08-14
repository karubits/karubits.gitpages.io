---
layout: post
title: "Deploying a Unifi Network Controller"
date: 2022-08-13 10:00:00 +0900
categories: network
tags: network unifi network controller ubiquiti ansible
---

# Unifi Network Controller and L3 Adoption

The Unifi Network Controller is a piece of software developed by Ubiquiti Inc, used for managing and administrating Unifi wireless access points, Unifi switches, and Unifi gateways.

In the guide we will cover:
- Installing the Unifi Network Controller with Ansible on bare metal / VM.

Its worth pointing out there are a few other options for running the Network Controller with baremetal / VM is not for you.

Some alternative options are:
1. Unifi Appliances - The [Unifi cloud key, dream machine, dream router](https://www.ui.com/consoles) has the Unifi Network Controller "App" built in. This is the fastest way to get up and running on Unifi's own hardware. You can use the Cloud Key Gen 2, or the Dream Machine
2. The team at [LinuxServer.io](https://www.linuxserver.io/) are maintaing a container of the Uniifi Network Controller to get up and running fast. [Dockerhub link](https://hub.docker.com/r/linuxserver/unifi-controller)
3. Cloud hosted Unifi Network Controller. The team over at Hostifi have dedicated managed cloud instances of the Unifi Network Controller. [Check them out here](https://www.hostifi.com/).


# Installation

The Unifi Network Controller has three main components

1. A database (MangoDB)
2. Java (In this case we will use AdoptJDK)
3. The applicaiton itself.

To get up and running as quickly as possible we will use Ansible to take care of the majority of tasks and dependcies for us.

> It should be noted at the current time of writting, the Unifi debian package will only accept MongoDB 3.x. However MongoDB 3.6 has been end-of-life since April 2021. I honestly hope that Unifi can update their MongoDB support as without it, it can present security risks or more concering would bring into question their commitment for hosting your own network controller and expect you to transition to their appliances. This is exactly what happened with the transisition from Unifi Video to Unifi Protect (an appliance only app).
{: .prompt-warning }


## Instalaltion Prerequisies

1. A Debian/Ubuntu based installation. In this guide we will use Debian Bullseye.
2. SSH access is configured and the server is reachable from the client you will run Ansible from.
3. Ansible is installed on your PC. 

## Getting Started

### Fast track (Ansible)

Check out my repositorty for an Ansible playbook that supports both Debian and Ubuntu to get and running with the Unifi Controller in minutes. I did the hard work for you and better yet Ansible helps to document that work.

The README.md should contain all the information you need to get started:

[Github - Karubits - Ansible Unifi Network Controller](https://github.com/karubits/ansible-unifi-network-controller)

### Tradational Approach

Lets do this long way then. As the Cloudkey and other appliances also run Debian, this guide will also just focus on Debian over Ubuntu. There is some logic behind this decision which is also described in the github link above. 


In a future article I will cover L3 (Layer 3) adoption, the benefits and how to approach adoption. Thank you for reading!

`TO BE CONTINUED`

