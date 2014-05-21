---
layout: post
title: "Using an SSH proxy to access your HP Helion Virtual Private Cloud instances"
description: ""
category: "howto"
tags: [hp, helion, openstack, vpc, ssh, proxy, jumphost, proxycommand, netcat]
published: true
---

## Using an SSH proxy to access your HP Helion Virtual Private Cloud instances

When you [sign up](https://horizon.hpcloud.com/register) for an [HP Helion](http://www.hpcloud.com) account, you will be setup with a basic Virtual Private Cloud (VPC).  This, in part, means that all of the instances you spin up inside this VPC will live in a private network and be inaccessible to your client (e.g. laptop) until you explicitly assign floating IP addresses to them, establish a site-to-site VPN between it (or your home network, for example) and your VPC, or use proxies.

If you are only (or mostly) interested in just ssh'ing to these instances from your laptop and don't want to fiddle with VPNs or using public addresses every time you, then proxying may be what you want and luckily a fairly simple solution exists that you may want to try thanks to OpenSSH, ProxyCommand, and netcat.

The steps for creating this SSH proxy (also known as a "jump host") are as follows.

### Assumptions
- You have an HP Helion account
- You have a personal SSH key registered with nova (referred to as MY_KEY_NAME / MY_KEY.pem below)

### Requirements

- Client (e.g. your laptop)
	- OpenSSH
	- python-novaclient
- Proxy / jump host
	- netcat

### Instructions

1. Install requirements on client

		$ sudo apt-get install -y openssh-client python-novaclient
    
2. Create an SSH keypair for your proxy / jump host

		$ nova keypair-add JUMPHOST_KEY_NAME > /path/to/JUMPHOST_KEY.pem

3. Create your proxy / jump host

		$ nova boot --image df3debd0-9391-4292-b4fe-fd3a700e7f4e 
		--flavor=standard.small --key-name=JUMPHOST_KEY_NAME "VPC Jump Host"

4. Create a floating IP address and associate it with the proxy / jump host

		$ nova floating-ip-create
		$ nova floating-ip-associate JUMPHOST_INSTANCE_ID JUMPHOST_FLOATING_IP

5. Install requirments on proxy / jump host

		$ ssh -i /path/to/JUMPHOST_KEY.pem ubuntu@JUMPHOST_FLOATING_IP sudo apt-get
		install -y netcat

6. Add / append the following proxy information to ~/.ssh/config on client

		Host 15.125.127.93
    		User ubuntu
    		IdentityFile /path/to/JUMPHOST_KEY.pem
    		ProxyCommand none

		Host 10.0.0.*
    		User ubuntu
    		IdentityFile /path/to/MY_KEY.pem
    		ProxyCommand ssh JUMPHOST_FLOATING_IP nc %h %p

7. Create a test instance

		$ nova boot --image df3debd0-9391-4292-b4fe-fd3a700e7f4e 
		--flavor=standard.small --key-name=MY_KEY_NAME "Test instance"
		$ nova list

8.  SSH into instance from client

	    $ ssh TESTINSTANCE_PRIVATE_IP_ADDRESS
