---
title: Azure ExpressRoute Initial Setup
description: Initial configuration of an Azure ExpressRoute Private Peering
slug: expressroute-initial-setup
date: 2025-10-20 00:00:00+0000
image: expressrouteIcon.png
categories:
    - Networking
tags:
    - Azure
    - ExpressRoute
    - Routing
    - BGP
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

Finding good documentation on the configurations needed for the CE side of an ExpressRoute setup is difficult. I recently went through an initial setup and ran into several obstacles. <b>This post will focus on the CE config.</b>

First, you need an ExpressRoute circuit established. This is done through a Microsoft partner. I'm assuming you're already done with this step and are ready to actually begin configuring things.

Next, you'll need to decide on two /30 networks to assign for the BGP peerings with Microsoft. Keep in mind, these two /30s must NOT overlap with anything you end up building in Azure. For example, if you choose 10.0.0.0/30 and 10.0.0.4/30, you CANNOT use 10.0.0.0/24 within a vnet later (exception: if the vnet containing the overlapping subnet is not and never will be peered to the vnet with the circuit within it. I'd just avoid overlapping altogether).

Once that's decided, configure them within Azure. It's very straightforward and there's plenty of documentation out there on this part. <a href="https://learn.microsoft.com/en-us/azure/expressroute/expressroute-howto-routing-portal-resource-manager" target="_blank">See here for Azure-side private peering configuration.</a>

On the customer equipment (CE) side, you're going to be configuring the BGP peer on your edge device. In the example below, it'll be configs for a Cisco ASR router. 

```
### Azure ExpressRoute CE Router BGP configuration ###
!
ip prefix-list AZURE_PREFIX_FILTER seq 10 permit 192.168.1.0/24
ip prefix-list AZURE_PREFIX_FILTER seq 20 permit 192.168.2.0/24
!
router bgp 64512
bgp log-neighbor-changes
neighbor 10.0.0.2 remote-as 12076
neighbor 10.0.0.2 ebgp-multihop 255
neighbor 10.0.0.6 remote-as 12076
neighbor 10.0.0.6 ebgp-multihop 255
!
address-family ipv4
  network 192.168.1.0/24
  network 192.168.2.0/24
  neighbor 10.0.0.2 activate
  neighbor 10.0.0.2 prefix-list AZURE_PREFIX_FILTER out
  neighbor 10.0.0.6 activate
  neighbor 10.0.0.6 prefix-list AZURE_PREFIX_FILTER out
exit-address-family
!
interface GigabitEthernet0/0/0.5
description ExpressRoute Connection
encapsulation dot1Q 5
ip address 10.0.0.5 255.255.255.252 secondary
ip address 10.0.0.1 255.255.255.252
End
```
A few key takeaways from this template:
- Make sure you include an outbound prefix list. If you are advertising too many prefixes to Azure, it will not form neighborships properly.
- Microsoft's ASN will always be 12076 for private peering.
- Your IP in the two /30 subnets will always be the first usable IP in the subnet. The second usable IPs need to be configured on the Azure side.
- the "ebgp-multihop" statement must be configured as your Azure neighbor will not be directly connected.
- If you're having trouble with the peering, you can verify your route and ARP tables on both your edge device and on the Azure side. Work on getting pings working from your subinterface IPs to your Azure peer IPs and work your way back.
- It's not shown in the template above, but be sure to add ACLs to your subinterface.
 
Importantly: READ THE BGP DEBUG LOGS. If you see SAFI/AFI errors, you'll need to make sure your address-family configs are correct and compatible with the peer. Keep most BGP settings default as there is not much documentation regarding specific Azure peer BGP settings. If you see logs saying the peer has reached its prefix limit, verify your outbound prefix lists are correct and filtering only what you want to advertise.
 

Useful BGP troubleshooting commands:
- Show ip bgp summary -- shows BGP neighbors and if they're exchanging routes
- Show ip bgp neighbor 10.0.0.2 -- shows detailed info about a BGP neighbor
- Debug ip bgp -- can specify a neighbor to debug so you don't get irrelevant logs
- Term mon -- to view BGP debug logs

Don't forget: your cloud IP space will need added to ENS software, firewall rules, router ACLs, and any other points in your infrastructure that may be filtering traffic!

Useful Links:
- <a href="https://learn.microsoft.com/en-us/azure/expressroute/" target="_blank">ExpressRoute Documentation</a>
- <a href="https://learn.microsoft.com/en-us/azure/expressroute/expressroute-config-samples-routing" target="_blank">ExpressRoute Router Configuration Sample</a>
