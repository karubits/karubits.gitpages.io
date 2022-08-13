---
layout: post
title: "Deploying a Unifi Network Controller and L3 Adoption"
date: 2022-08-13 10:00:00 +0900
categories: network
tags: network unfi network controller ubiquiti ansible
---
# DRAFT / WIP

# Unifi Network Controller and L3 Adoption

The Unifi Network Controller is a piece of software developed by Ubiquiti Inc, used for managing and administrating Unifi wireless access points, Unifi switches, and Unifi gateways.

In the guide we will cover:
- Install the Unifi Network Controller with Ansible on bare metal / VM.
- Explaining L3 adoption

Its worth pointing out there are a few other options for running the Network Controller with baremetal / VM is not for you.

Some alternative options are:
1. Unifi Appliances - The [Unifi cloud key, dream machine, dream router](https://www.ui.com/consoles) has the Unifi Network Controller "App" built in. This is the fastest way to get up and running on Unifi's own hardware. You can use the Cloud Key Gen 2, or the Dream Machine
2. The team at [LinuxServer.io](https://www.linuxserver.io/) are maintaing a container of the Uniifi Network Controller to get up and running fast. [Dockerhub link](https://hub.docker.com/r/linuxserver/unifi-controller)
3. Cloud hosted Unifi Network Controller. The team over at Hostifi have dedicated managed cloud instances of the Unifi Network Controller. [Check them out here](https://www.hostifi.com/).


## Installation

The Unifi Network Controller has three main components

1. A database (MangoDB)
2. Java (In this case we will use AdoptJDK)
3. The applicaiton itself.

To get up and running as quickly as possible we will use Ansible to take care of the majority of tasks and dependcies for us.


### Pre-requisies

1. A Debian/Ubuntu based installation. In this guide we will use Debian Bullseye.
2. SSH access is configured and the server is reachable from the client you will run Ansible from.
3. Ansible is installed on your PC. 

### Getting Started
