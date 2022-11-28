# Benefits of ExpressRoute cross-connections (bow-tie) in standard Hub&Spoke vs. VirtualWAN-based environments

In this article we will explore the benefits introduced by Expressroute cross-connections (bow-tie) for multi-region topologies in case of failure of 

1.	One Expressroute circuit 
2.	One entire Azure region

Note that with “failure of one ExpressRoute circuit” we here mean the failure of an entire peering location, so the loss of both links between primary and secondary MSEE routers.

Public doc source for this topic can be found [here](https://learn.microsoft.com/en-us/azure/expressroute/designing-for-disaster-recovery-with-expressroute-privatepeering#large-distributed-enterprise-network).

We will split the scenarios between 2 macro-areas.

The first will be focused on usage of standard HUB&Spoke topologies, so leveraging standard VNETs as HUB.
The second will report the differences introduced by the usage of Azure VirtualWAN.

All the scenarios are based on a pair of Azure regions linked (in different ways) to a pair of Onprem branches / DCs.
The 2 Onprem DCs are here supposed to be connected by customer MPLS/WAN backbone, but we will mention the implications of circuit’s/region’s failures for isolated branches as well.

# **Standard Hub and Spoke environments**

## **SCENARIO 1 – Without bow-tie connections**

<img src=Pics/H%26S%20-%20no%20bowtie%20-%20general.jpg />

### **Failure of ExpressRoute circuit**

<img src=Pics/H%26S%20-%20no%20bowtie%20-%20circuitfail.jpg width="700" height="500" align="center" />

In case of failure of C1:

-	The access to region A will be lost from any branch
-	Onprem branches will still be able to access Region B leveraging customer’s MPLS/WAN backbone
-	When both circuits are available, the 2 Azure regions could here potentially connect by leveraging customer’s MPLS backbone, but with failure of C1 the 2 regions can’t connect.

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a circuit will here determine lack of connectivity with any Azure region. Branch2Branch transit would not be possible here, even with both circuits up.
