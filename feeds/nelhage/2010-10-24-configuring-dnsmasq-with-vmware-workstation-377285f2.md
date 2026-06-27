---
title: Configuring dnsmasq with VMware Workstation
url: https://blog.nelhage.com/2010/10/dnsmasq-and-vmware/
published: "2010-10-24T23:15:23Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/10/dnsmasq-and-vmware/
---

# Configuring dnsmasq with VMware Workstation

I love VMware workstation. I keep VMs around for basically every version of every major Linux distribution, and use them heavily for all kinds of kernel testing and development. This post is a quick writeup of my networking setup with VMware Workstation, using dnsmasq to assign my VMs addresses and provide a DNS server to resolve VM addresses. The objective I want to be able to resolve my VM’s hostnames so that I can ssh to them, or run other network services and access them from the host.
