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
- Install the Unifi Network Controller with Ansible
- Explaining L3 adoption

## Installation

The Unifi Network Controller has three main components

1. A database (MangoDB)
2. Java (In this case we will use AdoptJDK)
3. The applicaiton itself.


### Pre-requisies

1. A Debian/Ubuntu based installation. In this guide we will use Debian Bullseye.

### Getting Started


