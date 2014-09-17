---
layout: post
title: Vagrant Up
tags: [vagrant, centos, virtualbox]
---

I've been using [Vagrant](https://vagrantup.com) since the beginning of the year, and I'm 
habitually amazed by how easy it is. I find it as easy as using `git`, and 
think both `git` and `vagrant` are amazing tools that make life as a developer
easier. Once you have your properly configured Vagrant box, you are good to go. 

But therein lies the rub. I'm suspicious of 
boxes hosted in Dropbox accounts, and have therefore been limited to running 
the default Ubuntu boxes although lately I use CentOS for development almost
exclusively. So I set out to create an extremely basic CentOS box from 
scratch, took copious notes along the way, and now that I have declared success 
I'm ready to share what I learned.

## Assumptions
I have made every attempt to make this walkthrough as clear as possible, but 
I made some assumptions about people trying to follow this tutorial which may
or may not apply to you. But if you're uncomfortable, say, using `vi`, feel 
free to jump to your favorite 
terminal editor to get things done; I trust your judgment in making such 
substitutions. Another assumption is that you are using a Windows machine but are
sufficiently comfortable running bash commands. The steps outlined here were 
made in a Windows 7 x64 machine; they should be valid on more recent Windows 
installations as well.

## Getting Started
### Install git [Optional]
If you do not have git installed, head over to [git-scm.com](https://git-scm.com)
and install it. Once installed, add the `/path/to/Git/bin` directory to your Path
System Variable. This will allow you to SSH to your guest machine without the
need for a dedicated SSH client like PuTTy.

### Install VirtualBox, download CentOS
Download and install [VirtualBox](https://www.virtualbox.org), then head over 
to the [CentOS Project Download page](https://www.centos.org/download/), and 
grab a 64bit version of the OS. I used CentOS 6.4 x86_64 minimal (no GUI) for my environment.

## Create VirtualBox Appliance and Install CentOS
First you need to create a regular virtual machine in VirtualBox, which 
you will later export as a Vagrant box.

Fire up VirtualBox and click on New. When asked to provide a name for your 
machine choose something sensible, like centos65_64bit. Give it 
the desired amount of RAM and the desired virtual hard drive. Once it is 
created, right-click the VM and then left-click on Settings.

On the Settings screen, go to Storage. In the 'Storage Tree' section, click 
on the CD/DVD drive icon and another CD/DVD icon should appear on your right.
Click it and choose the CentOS ISO that you downloaded earlier.

Still on the settings screen, click on Network. Your Adapter 1 should be NAT
and be enabled by default. Expand the Advanced options for this adapter, then
click on Port Forwarding. Add a new Port Forwarding rule here, giving it a name
'guestssh', make sure that the protocol is TCP, the Host Port 2222 and the Guest
Port 22. Click OK once this is done, leave the IP addresses blank.

The last step in the Settings screen is to go to the USB section and uncheck
'Enable USB Controller'. Click OK on the Settings screen to save all this.

Your VM is now ready to install CentOS from the ISO, so start it up. I confess
that I headed straight for an install and skipped pretty much all checks.
Configure the boring stuff like Timezone, language, etc., but when prompted for
a root password, use the word `vagrant`. Then click next until it starts installing
the OS. When it's done, it should prompt you to reboot, and you should do so.
