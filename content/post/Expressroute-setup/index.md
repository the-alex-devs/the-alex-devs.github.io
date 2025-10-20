---
title: Azure ExpressRotue Initial Setup
description: Initial configuration of an Azure ExpressRoute Private Peering
slug: expressroute-initial-setup
date: 2025-10-20 00:00:00+0000
image: cover.jpg
categories:
    - Networking
tags:
    - Azure
    - Cloud
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

Finding good documentation on the configurations needed for the CE side of an ExpressRoute setup is difficult. I recently went through an initial setup and ran into several obstacles. <b>This post will focus on the CE config.</b>

First, you need an ExpressRoute circuit established. This is done through a Microsoft partner. I'm assuming you're already done with this step and are ready to actually begin configuring things.

Next, you'll need to decide on two /30 networks to assign for the BGP peerings with Microsoft. Keep in mind, these two /30s must NOT overlap with anything you end up building in Azure. For example, if you choose 10.0.0.0/30 and 10.0.0.4/30, you CANNOT use 10.0.0.0/24 within a vnet later (exception: if the vnet containing the overlapping subnet is not and never will be peered to the vnet with the circuit within it. I'd just avoid overlapping altogether).

Once that's decided, configure them within Azure. It's very straightforward and there's plenty of documentation out there on this part. <a href="https://learn.microsoft.com/en-us/azure/expressroute/expressroute-howto-routing-portal-resource-manager" target="_blank">See here for Azure-side private peering configuration.</a>

TO BE CONTINUED




Useful Links:

<a href="https://learn.microsoft.com/en-us/azure/expressroute/" target="_blank">ExpressRoute Documentation</a>


