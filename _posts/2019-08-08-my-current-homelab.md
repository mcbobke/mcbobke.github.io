---
title: 'My Current Homelab'
date: 2019-08-05
permalink: /2019/08/08/my-current-homelab/
categories:
  - Homelab
tags:
  - homelab
---
I have been married to my wife Amanda since March of this year and part of that process included us moving in to our first apartment together. I took the opportunity to start my first homelab after promising Amanda that maintenance windows would occur after she went to bed and the Wi-Fi would be up before morning.

I think homelabs are a vital part of education in any tech career path and mine continues to play a huge role in my personal learning. Unfortunately it's difficult to represent a homelab on a resume so this post is the start of an effort to document mine in its current state with future update posts to come when major changes occur.

## Network Diagram

Here's my first attempt at a network diagram in Visio.

![Network Diagram](/images/2019-08-my-current-homelab/HomeNetwork.png)

I admit that I underestimated both the vastness of Visio's featureset and its complexity. I spent an embarassing amount of time trying to get custom-drawn line connectors between a few of the shapes and eventually gave up in favor of straight lines for the time being. I'm sure if/when the diagram grows larger that I will need to spend more time perfecting clean connectors.

Another frustration I had was the lack of a generic "legend" shape beyond that of the auto-generated legend shape available under the "Networks and Peripherals" category. This included legend is not very customizable. Some example diagrams that I've seen on reddit (shout out to [/r/homelab](https://reddit.com/r/homelab)) have fairly extensive shape legends that explain what each type of shape on the diagram represents. These must be hand-crafted and for both my small diagram and the majority of those of similar size on /r/homelab, the only legend that I ultimately included is one detailing VLAN representation by wire color.

I like that Visio is forgiving about using images in place of custom shapes when they aren't available, but thankfully many vendors recognize that Visio is a standard in the diagram and flowchart space and consequently provide their own shapes.

Lets go over the diagram piece by piece.

## EdgeRouter X and Unifi AP-AC-Lite

I did a fair bit of research into home networking equipment appropriate for the average apartment and the combination of an [EdgeRouter X](https://www.ui.com/edgemax/edgerouter-x/) and an [Unifi AP-AC-Lite](https://www.ui.com/unifi/unifi-ap-ac-lite/) came up much more than once. I have been extremely happy with the performance of the two given that we need to run almost every device in our apartment on Wi-Fi. I was worried that I would notice a huge drop in network performance coming from my previous home that had Ethernet drops in every room but our network speeds are consistently between 150-200 Mbps download and 25-30 Mbps upload.

Setup of both of these devices was necessarily more complicated than that of your average plug-and-play router/AP combo device but it's well worth the expanded featureset if you're looking to learn more about enterprise-grade networking equipment configuration. I followed [Crosstalk Solutions' video](https://www.youtube.com/watch?v=SianDqAQaR0) for the EdgeRouter X and [Ubiquiti's quick-start guide](https://dl.ubnt.com/guides/UniFi/UniFi_AP-AC-Lite_QSG.pdf) for the AP-AC-Lite.

So far, you can see in the network diagram that three of the interfaces are in use on the EdgeRouter X: eth0 for WAN, eth1 to a Netgear 8-port unmanaged L2 switch, and eth2 to the AP-AC-Lite. I don't have a need to use eth3 yet; eth4 is the PoE passthrough port on this router and if I buy a stronger power supply for it I can use that port to provide power to the AP-AC-Lite. I'd like to do that to remove the need for the PoE injector that comes with the AP-AC-Lite for cleaner cabling. I'm not sure why the device doesn't come with a power supply with a high enough wattage to support PoE (included power supply is 6W, needs 12W for PoE) but thankfully it's an inexpensive upgrade.

The EdgeRouter X includes an L3 switching chip that allows the interfaces on the router to be grouped into a switch configuration (switch0 on the following image) which means that VLANs can be created. I had no idea what I was doing the first time I made an attempt at segregating traffic between two VLANs and quickly learned about the importance of regular config backups in the event of a misconfiguration that locks you out of your router. After a couple hours of rebuilding my router and all of its static DHCP mappings I sat down and poured over documentation before making any further moves (and made an easily-accessible config backup). Ubiquiti's documentation pretty much all but assumes that you have an understanding of higher-level networking concepts before providing examples of configuring their devices. This was assuming too much in my situation so it took me some time to figure out exactly what I needed to do. Ultimately I figured it out and this is what my interface configuration looks like now.

![Interface Configuration](/images/2019-08-my-current-homelab/switch_interface_config_08_19.png)

Each interface on switch0 has a PVID and VID value associated with it, PVID being the untagged network and VID being tagged. I only tagged the guest VLAN ID of 20 on the interfaces that I will be using for the AP-AC-Lite since it can tag traffic on individual networks that it broadcasts.

![Interface PVID/VID Configuration](/images/2019-08-my-current-homelab/switch_interface_tags_08_19.png)

On the Unifi Controller I have the guest network set with a VLAN ID of 20 so that the SSID is associated with the correct DHCP server and firewall rules.

![Unifi Controller Networks](/images/2019-08-my-current-homelab/unifi_networks_08_19.png)

Finally, I have a few firewall rulesets defined for the guest VLAN and the WAN interface. Any DNS traffic coming in from the guest network that is destined for the Pi-hole is accepted, all other traffic destined for the private VLAN is dropped, and all other traffic is accepted. Any traffic from the guest network to the local device (management access to the default gateway) is also dropped.

![Guest-In Rules](/images/2019-08-my-current-homelab/guest_in_rules_08_19.png)

On eth0, all established/related traffic both in and to the default gateway is accepted; all invalid state and any other traffic is dropped by default.

![WAN-In Rules](/images/2019-08-my-current-homelab/wan_in_rules_08_19.png)

There are two DHCP scopes set up on the EdgeRouter X, one for each VLAN. The Private VLAN scope uses my Active Directory DNS servers (the domain controllers) which are then forwarded to the Pi-hole. The Guest VLAN uses the Pi-hole directly.

![Private VLAN DHCP Settings](/images/2019-08-my-current-homelab/private_dhcp_08_19.png)
![Guest VLAN DHCP Settings](/images/2019-08-my-current-homelab/guest_dhcp_08_19.png)

That's about all that's interesting about my networking equipment.

## VM Host

My VM Host machine is an old gaming rig that I threw some spare drives into. It's got plenty of headroom for my purposes and has been able to handle anything that I've thrown at it using an i7-3770k and 32GB of DDR3.

I'm running ESXi 6.7u2 as the hypervisor which is installed on a 32GB USB thumb drive. That way a loss of the OS drive won't take out any of the VMs. I started with ESXi 6.5 which was the current version when I originally installed it. I managed to successfully update the OS using [this guide](https://tinkertry.com/easy-update-to-esxi-67) but I encountered a strange error about a lack of available disk space which was solved by the solution [found here](https://communities.vmware.com/thread/595325).

The VMs that I'm running are:

* Windows Server 2016 Datacenter - Primary DC/DNS
* Windows Server 2016 Datacenter - Secondary DC/DNS
* Ubuntu Server 18.04 - Nagios Monitoring Server
* Ubuntu Server 18.04 - Pi-hole
* Ubuntu Server 18.04 - Unifi Controller

The two domain controllers are not the first that I've ever set up in my lab environment, but this time I was determined to do it correctly. Previously I did not have control over the DNS servers on my network but now that I can route DNS however I choose, I can connect computers to the domain without manually setting DNS servers on each computer. I also configured a GPO for [BGInfo](https://docs.microsoft.com/en-us/sysinternals/downloads/bginfo) to be run automatically on every server.

![BGInfo on Primary Domain Controller](/images/2019-08-my-current-homelab/bginfo.png)

Setting up Nagios for monitoring my Linux boxes proved to be a bit more involved than expected since I needed to build from source, but that wasn't very complicated with the documentation. I've managed to implement monitoring of CPU usage, memory usage, and simple service availability (SSH being allowed, for example) on my Linux machines and want to bring my Windows machines into scope using [NSClient ++](https://www.nsclient.org/) soon.

When I first set up my networking equipment I installed the Unifi Controller software on my laptop. I didn't realize that this is a service that's designed to be running at all times and it quickly proved to be inconvenient to use on my laptop. Ubiquiti provides a guide [here](https://help.ubnt.com/hc/en-us/articles/220066768-UniFi-How-to-Install-and-Update-via-APT-on-Debian-or-Ubuntu) that details how to correctly set up the Unifi Controller on a Linux machine. I backed up my config from my laptop and imported it onto the new VM which was easy and solved the problem.

Most of the VMs and ESXi itself aren't very exciting to look at, but the VM that I think is very interesting is the one running Pi-hole. As a result of implementing it I've discovered just how much of our everyday internet traffic is junk/ad traffic:

![Pi-hole Chart](/images/2019-08-my-current-homelab/pi-hole-08-19.png)

Minutes after I routed DNS traffic through the Pi-hole Amanda came to me and asked why the ads on some social media apps weren't loading. I'm seriously impressed by its performance and it was a simple service to stand up.

## Synology DS718+

This is probably my favorite device in my homelab. I was looking to buy this exact model brand new on Amazon along with two 4TB WD Red drives which would have totaled around $700 after all was said and done. I decided to check eBay before I purchased it and I found the exact same model hardly used with two 4TB Seagate Ironwolf drives inside for $400 shipped. I jumped on that and when it arrived the S.M.A.R.T. readings on the drives showed them as being used for less than 200 hours total. Its single defect is one of the plastic clips that holds the back part of the chassis on is broken so I had to hold it down with a piece of electrical tape; I'll take that for $300 in savings!

I use this NAS for storing personal files that I want stored redundantly but don't feel comfortable storing in the cloud. I have the drives configured in Synology's Hybrid RAID which is essentially just a mirror. I have Plex configured as a service on this device which has run very well despite not having much CPU power or RAM to work with.

## Future Plans

That was a quick overview of the interesting parts of my homelab. Now that I'm writing a ton of DSC at work I think I'll stand up a few more Windows Server VMs for testing DSC configs against them. I also want to deploy a few more Ubuntu VMs and get started with Linux configuration management tools like Ansible, Puppet, and Chef. I also want to try and get PXE booting working for VM deployment to help speed up the process; installing VMs manually from beginning to end is not very great after the second or third time through. Additionally I hope to add an 8-port EdgeSwitch in the future to get experience using a managed switch, but thankfully the capabilities of the EdgeRouter X have already exposed me to many concepts beyond those of a regular router.
