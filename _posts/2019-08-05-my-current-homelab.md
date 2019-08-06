---
title: 'My Current Homelab'
date: 2019-08-05
permalink: /2019/08/05/my-current-homelab/
categories:
  - Homelab
tags:
  - homelab
---
I have been married to my wife Amanda since March of this year and part of that process included us moving in to our first apartment together. I took the opportunity to start my first homelab after promising Amanda that maintenance windows would occur after she went to bed and the Wi-Fi would be up before morning.

I think homelabs are a vital part of education in any tech career path and mine continues to play a huge role in my personal learning. Unfortunately it's difficult to represent a homelab on a resume so this post is the start of an effort to document mine in its current state with future update posts to come when major changes occur.

## Network Diagram

Here's my first attempt at a network diagram in Visio.

![Network Diagram](/images/HomeNetwork.png)

I admit that I underestimated both the vastness of Visio's featureset and its complexity. I spent an embarassing amount of time trying to get custom-drawn line connectors between a few of the shapes and eventually gave up in favor of straight lines for the time being. I'm sure if/when the diagram grows larger that I will need to spend more time perfecting clean connectors.

Another frustration I had was the lack of a generic "legend" shape beyond that of the auto-generated legend shape available under the "Networks and Peripherals" category. This included legend is not very customizable. Some example diagrams that I've seen on reddit (shout out to [/r/homelab](https://reddit.com/r/homelab)) have fairly extensive shape legends that explain what each type of shape on the diagram represents. These must be hand-crafted and for both my small diagram and the majority of those of similar size on /r/homelab, the only legend that I ultimately included is one detailing VLAN representation by wire color.

I like that Visio is forgiving about using images in place of custom shapes when they aren't available, but thankfully many vendors recognize that Visio is a standard in the diagram and flowchart space and consequently provide their own shapes.

Lets go over the diagram piece by piece.

## EdgeRouter X and Unifi AP-AC-Lite

I did a fair bit of research into home networking equipment appropriate for the average apartment and the combination of an [EdgeRouter X](https://www.ui.com/edgemax/edgerouter-x/) and an [Unifi AP-AC-Lite](https://www.ui.com/unifi/unifi-ap-ac-lite/) came up much more than once. I have been extremely happy with the performance of the two given that we need to run almost every device in our apartment on Wi-Fi. I was worried that I would notice a huge drop in network performance coming from my previous home that had Ethernet drops in every room but our network speeds are consistently between 150-200 Mbps download and 25-30 Mbps upload.

Setup of both of these devices was necessarily more complicated than that of your average plug-and-play router/AP combo device but it's well worth the expanded featureset if you're looking to learn more about enterprise-grade networking equipment configuration. I followed [Crosstalk Solutions' video](https://www.youtube.com/watch?v=SianDqAQaR0) for the EdgeRouter X and [Ubiquiti's quick-start guide](https://dl.ubnt.com/guides/UniFi/UniFi_AP-AC-Lite_QSG.pdf) for the AP-AC-Lite.

So far, you can see in the network diagram that three of the interfaces are in use on the EdgeRouter X: eth0 for WAN, eth1 to a Netgear 8-port unmanaged L2 switch, and eth2 to the AP-AC-Lite. I don't have a need to use eth3 yet; eth4 is the PoE passthrough port on this router and if I buy a stronger power supply for it I can use that port to provide power to the AP-AC-Lite. I'd like to do that to remove the need for the PoE injector that comes with the AP-AC-Lite for cleaner cabling. I'm not sure why the device doesn't come with a power supply with a high enough wattage to support PoE (included power supply is 6W, needs 12W for PoE) but thankfully it's an inexpensive upgrade.

The EdgeRouter X includes an L3 switching chip that allows the interfaces on the router to be grouped into a switch configuration (switch0 on the following image) which means that VLANs can be created. I had no idea what I was doing the first time I made an attempt at segregating traffic between two VLANs and quickly learned about the importance of regular config backups in the event of a misconfiguration that locks you out of your router. After a couple hours of rebuilding my router and all of its static DHCP mappings I sat down and poured over documentation before making any further moves (and made an easily-accessible config backup). Ubiquiti's documentation pretty much all but assumes that you have an understanding of higher-level networking concepts before providing examples of configuring their devices. This was assuming too much in my situation so it took me some time to figure out exactly what I needed to do. Ultimately I figured it out and this is what my interface configuration looks like now.

![Interface Configuration](/images/switch_interface_config_08_19.png)

Each interface on switch0 has a PVID and VID value associated with it, PVID being the untagged network and VID being tagged. I only tagged the guest VLAN ID of 20 on the interfaces that I will be using for the AP-AC-Lite since it can tag traffic on individual networks that it broadcasts.

![Interface PVID/VID Configuration](/images/switch_interface_tags_08_19.png)

On the Unifi Controller I have the guest network set with a VLAN ID of 20 so that the SSID is associated with the correct DHCP server and firewall rules.

![Unifi Controller Networks](/images/unifi_networks_08_19.png)

Finally, I have a few firewall rulesets defined for the guest VLAN and the WAN interface. Any DNS traffic coming in from the guest network that is destined for the PiHole is accepted, all other traffic destined for the private VLAN is dropped, and all other traffic is accepted. Any traffic from the guest network to the local device (management access to the default gateway) is also dropped.

![Guest-In Rules](/images/guest_in_rules_08_19.png)

On eth0, all established/related traffic both in and to the default gateway is accepted; all invalid state and any other traffic is dropped by default.

![WAN-In Rules](/images/wan_in_rules_08_19.png)
