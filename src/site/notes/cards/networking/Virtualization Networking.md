---
{"dg-publish":true,"permalink":"/cards/networking/virtualization-networking/"}
---

[[atlas/networking\|networking]]

Created: 2023-08-13

Pre-requisite:
- [[Network Address Translation\|Network Address Translation]]
- [[DHCP\|DHCP]]

The most common networking mode used:

- NAT mode
- [[#NAT network|Virtualization Networking]]
- [[#Intnet|Virtualization Networking]] (Internal Network)
- [[# Bridge Adapter|Virtualization Networking]]

> [!importat]+ The IP address provided in the images are the defaults
## NAT MODE

![Virtualization Networking.png](/img/user/cards/networking/images/Virtualization%20Networking.png)

In this mode:

![Virtualization Networking-1.png](/img/user/cards/networking/images/Virtualization%20Networking-1.png)

- VM to VM is not possible because in this mode every VM is isolated
- VM to host or any device in the LAN is possible as the host acts as the Internet where the virtual NAT translates the VM IP address into the host [[IP address\|IP address]] class
- [[Port Forwarding\|Port Forwarding]] is required if devices from the physical LAN wants to access services from the VM.

> [!question]- What will be the process if VM wants to access the Internet?
> VM -> Virtual NAT -> host address -> NAT -> Internet
## NAT network

![Virtualization Networking-2.png](/img/user/cards/networking/images/Virtualization%20Networking-2.png)

In this mode:

![Virtualization Networking-3.png](/img/user/cards/networking/images/Virtualization%20Networking-3.png)

It is almost the same as the previous mode but the difference is there is a virtual switch that allows the VM to communicate with each other, essentially it is like our home network.
## Intnet

![Virtualization Networking-4.png](/img/user/cards/networking/images/Virtualization%20Networking-4.png)

In this mode:

![Virtualization Networking-5.png](/img/user/cards/networking/images/Virtualization%20Networking-5.png)

An isolated network that cannot access the physical LAN and vice versa, also we can create more than 'intnet' if we want to simulate more isolated networks.
## Bridge Adapter

![Virtualization Networking-6.png](/img/user/cards/networking/images/Virtualization%20Networking-6.png)

In this mode:

![Virtualization Networking-7.png](/img/user/cards/networking/images/Virtualization%20Networking-7.png)

It is like **NAT network** mode but this time the VMs can be imagined as real devices connected to your Wi-fi or home network, IP address are assigned by our DHCP server (the router)

## Host-Only
---
Only permits network operation with the Host OS - no other computers running on the same network can access it.


![Virtualization Networking-8.png](/img/user/cards/networking/images/Virtualization%20Networking-8.png)

#### Resources


