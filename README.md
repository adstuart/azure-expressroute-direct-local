# Combine ExpressRoute Direct and ExpressRoute Local

<contents>

# 1. Introduction

## ExpressRoute Direct 

Many customers on Azure leverage ExpressRoute for reliable hybrid connectivity. Sometimes these same customers have requirements that make  the [ExpressRoute Direct](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-erdirect-about) [connectivity model](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-connectivity-models) the correct design choice. Typical drivers for ExpressRoute Direct include:

- Lowest possible latency
- Regulation or security requirements
- Hybrid connectivity bandwidth in excess of 10Gbps (Up to 100Gbps)
- Attachment of complicated multi-VRF MPLS IPVPN/WAN
- Simplification of connectivity design, logistics and commercial agreements

## ExpressRoute Local

As per [here](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-faqs#what-is-expressroute-local)

> ExpressRoute Local is a SKU of ExpressRoute circuit, in addition to the Standard SKU and the Premium SKU. A key feature of Local is that a Local circuit at an ExpressRoute peering location gives you access only to one or two Azure regions in or near the same metro.... 

> ...While you need to pay egress data transfer for your Standard or Premium ExpressRoute circuit, you don't pay egress data transfer separately for your ExpressRoute Local circuit. In other words, the price of ExpressRoute Local includes data transfer fees.

## Better together

In this article we will show how its possible combine these two ExpressRoute products, Local and Direct, and align with the common "Bowtie" [design pattern](https://docs.microsoft.com/en-us/azure/expressroute/designing-for-disaster-recovery-with-expressroute-privatepeering#large-distributed-enterprise-network) used by the majority of enterprise customers on Azure. 

This combination is, at first, not obvious, and hence this article was created. It shows how we able to maintain a highly resilient design, whilst benefiting from the pricing model of ExpressRoute Local to reduce egress costs. This may or may not be the right solution for your company, it depends on how much value you will derive from the pricing discounts, vs the downsides of the technical considerations laid on herein.

> **Note! The use of ExpressRoute local is particular of interest when utilising ExpressRoute Direct due to the fact that Local and Standard circuits are included in the base ExpressRoute Direct [port costs](https://azure.microsoft.com/en-gb/pricing/details/expressroute/).**

# Technical Design

## Traditional Bowtie topology

The following diagram shows how the typical Enterprise bowtie design would be deployed to mesh two regions with two resilient ExpressRoute peering locations. This is the model used by most Enterprise customers using the partner connectivity models, and can be equally implemented within the framework of ExpressRoute Direct. This is a valid connectivity pattern, and is the right choice for some scenarios, see technical considerations section.

The commercial impact of course is that these are standard ExpressRoute circuits, and therefore any data leaving Azure towards On-Premises incurs costs as per _Outbound Data Transfer pricing_ [here](https://azure.microsoft.com/en-gb/pricing/details/expressroute/).

![](images/2021-09-02-15-08-02.png)

## Modified Direct/Local Bowtie topology

Leveraging the logical, circuit, to physical, port, framework of ExpressRoute Direct we are able to deploy additional circuits both quickly and without commercial impact to achieve the design below.

![](images/2021-09-02-15-17-44.png)

This delivers upon the same expected uptime of the traditional bowtie design, with the added benefit of free data egress when leaving Azure towards On-Premises via a green ExpressRoute Local circuit.

The important call-out here (outside of the technical considerations laid out later), is that **standard SKU ExpressRoute circuits as still used for connectivity to remote regions**. (Remember that Local SKU circuits can [only be used to reach the Local Azure region](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-faqs#what-features-are-available-and-what-are-not-on-expressroute-local)). However, with specific traffic engineering, these circuits are configured to be only used during a failure scenario.

> Good to know. Do you need to know which Azure region is classified as the "local" region for a specific peering location? Checkout the [ExpressRoute partners and peering locations documentation page](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-locations-providers).  Notice the _Local Azure regions_ column [here](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-locations-providers#global-commercial-azure).

-

## Traffic Engineering

>> jeremies article

Automatic failover

## Net result

Resilient design, bau no egress cost, failover automatic and costs

## Cosiderations

GlobalReach
ER Hairpin
Microsoft Peering
Gateway limits
Circuit limits
Not premium for simplicity
peering location as to be local, metro design would not work
does not have to be all or nothing, logical circuits an mean differnet approach, er local for specific workload


# worked example

This guides suggest an approach to this migration process that focuses on seamless failover, de-risking rollback, and understanding the correct ordering of steps. Each step will require you to leverage your existing knowledge of ExpressRoute, links will be provided to Azure documentation as required for further technical depth. 

> :warning: This document assumes pre-existing knowledge of Azure, ExpressRoute and BGP. It is not designed to be read in isolation, but rather act as a high level guide, pointing you in the right direction, at the right places and get the project team asking the right questions to plan for success.

