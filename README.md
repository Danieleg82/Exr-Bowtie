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

### **1.A Failure of ExpressRoute circuit**

<img src=Pics/H%26S%20-%20no%20bowtie%20-%20circuitfail.jpg width="650" height="500" align="center" />

In case of failure of C1:

-	The access to region A will be lost from any branch
-	Onprem branches will still be able to access Region B leveraging customer’s MPLS/WAN backbone
-	When both circuits are available, the 2 Azure regions could here potentially connect by leveraging customer’s MPLS backbone, but with failure of C1 the 2 regions can’t connect.

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a circuit will here determine lack of connectivity with any Azure region. Branch2Branch transit would not be possible here, even with both circuits up.

<img src=Pics/H%26S%20-%20no%20bowtie%20-%20circuitfail2.jpg width="650" height="500" align="center" />

### **1.B Failure of a region**

<img src=Pics/H%26S%20-%20no%20bowtie%20-%20regionfail.jpg width="650" height="500" align="center" />

For this scenario, same considerations as per circuit's failure apply

## **SCENARIO 2: With bow-tie connections**

<img src=Pics/H%26S%20-%20bowtie%20-%20general.jpg />

### **2.A Failure of ExpressRoute circuit**

<img src=Pics/H%26S%20-%20bowtie%20-%20circuitfail.jpg width="650" height="500" align="center" />

In case of failure of C1:

-	Both regions will still be accessible from both branches, but Branch 1 will have to leverage customer WAN/MPLS to access them
-	Connectivity between Branch1 and RegionA will of course suffer of a lot of additional latency (traffic will have to travel cross-region through Branch2 --> reach C2 peering location --> travel back to RegionA via MS backbone --> and return)
-	The 2 Azure regions will still have connectivity between themselves via C2 circuit


**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a circuit will here determine lack of connectivity with any Azure region. Branch2Branch transit would not be possible here, even with both circuits up.

<img src=Pics/H%26S%20-%20bowtie%20-%20circuitfail2.jpg width="650" height="500" align="center" />

### **2.B Failure of a region**

<img src=Pics/H%26S%20-%20bowtie%20-%20regionfail.jpg width="650" height="500" align="center" />

In case of failure of entire Region A:

-	Region B will still be accessible by both branches.
-	Branch1 could access Region B leveraging the peering with C1 and MS backbone, rather than using customer’s MPLS backbone, with 2 big advantages:
    -	Customer’s MPLS backbone will be offloaded by such traffic
    -	The impact on latency, since we’re leveraging MS backbone, is expected to be optimized

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a region, with circuit (C1 here) still up & running , would still allow to have Branch1 connected to RegionB thanks to the cross-connection. Branch2Branch transit would not be possible here, even with both circuits up.

<img src=Pics/H%26S%20-%20bowtie%20-%20regionfail2.jpg width="650" height="500" align="center" />

# **Virtual WAN - based environments**

## **SCENARIO 3 – Without bow-tie connections**

<img src=Pics/vWAN%20-%20no%20bowtie%20-%20general.jpg />

### **3.A Failure of ExpressRoute circuit**

<img src=Pics/vWAN%20-%20no%20bowtie%20-%20circuitfail.jpg width="650" height="500" align="center" />

In case of failure of C1:

- Both regions will still be accessible by any branch!!
- Branch1 will be able to access Azure regions only leveraging customer’s MPLS backbone, with relevant impact on MPLS BW utilization and latency

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a circuit will here determine lack of connectivity with any Azure region. Branch2Branch transit would not be possible here, even with no circuit’s/region’s failure (with vWAN, a similar connection would be possible today only by leveraging ExpressRoute GlobalReach functionalities)

<img src=Pics/vWAN%20-%20no%20bowtie%20-%20circuitfail2.jpg width="650" height="500" align="center" />

### **3.B Failure of a region**

<img src=Pics/vWAN%20-%20no%20bowtie%20-%20regionfail.jpg width="650" height="500" align="center" />

In case of failure of entire Region A:

- Region B will still be accessible by both branches.
- Branch1 could access Region B only via customer’s MPLS backbone, with relevant impact on MPLS BW utilization and latency

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a region, with circuit (C1 here) still up & running , would still cause lack of connectivity with any Azure region. . Branch2Branch transit would not be possible here, even with no circuit’s/region’s failure (with vWAN, a similar connection would be possible today only by leveraging ExpressRoute GlobalReach functionalities)

<img src=Pics/vWAN%20-%20no%20bowtie%20-%20regionfail2.jpg width="650" height="500" align="center" />

## **SCENARIO 4: With bow-tie connections**

<img src=Pics/vWAN%20-%20bowtie%20-%20general.jpg />

### **4.A Failure of ExpressRoute circuit**

<img src=Pics/vWAN%20-%20bowtie%20-%20circuitfail.jpg width="650" height="500" align="center" />

In case of failure of C1:

- Both regions will still be accessible from both branches 
- Branch1 will be able to access Azure regions only leveraging customer’s MPLS backbone
- The 2 Azure regions will still have connectivity via MS backbone or C2 circuit, depending on the HUB routing preference configured 

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a circuit will here determine lack of connectivity with any Azure region. Branch2Branch transit would not be possible here, even with no circuit’s/region’s failure (with vWAN, a similar connection would be possible today only by leveraging ExpressRoute GlobalReach functionalities)

<img src=Pics/vWAN%20-%20bowtie%20-%20circuitfail2.jpg width="650" height="500" align="center" />

### **4.B Failure of a region**

<img src=Pics/vWAN%20-%20bowtie%20-%20regionfail.jpg width="650" height="500" align="center" />

In case of failure of entire Region A:

- Region B will still be accessible by both branches.
- Branch1 could access Region B leveraging the peering with C1 and MS backbone, rather than using customer’s MPLS backbone, with 2 big advantages:
    - Customer’s MPLS backbone will be offloaded by such traffic
    - The impact on latency – since we’re leveraging MS backbone – is expected to be optimized

**In case of isolated branches (with no WAN/MPLS connectivity)**: the loss of a region, with circuit (C1 here) still up & running , would still allow to have Branch1 connected to RegionB thanks to the cross-connection. . Branch2Branch transit would not be possible here, even with no circuit’s/region’s failure (with vWAN, a similar connection would be possible today only by leveraging ExpressRoute GlobalReach functionalities)

<img src=Pics/vWAN%20-%20bowtie%20-%20regionfail2.jpg width="650" height="500" align="center" />

# **Summary Tables**

## ***Circuit failure***

| | **Can I still reach primary region?**  | **Can I reach secondary region?** | **Can I reach Azure without leveraging my MPLS?** | **Can I have connectivity between primary and secondary regions?** | 
| ------------- | ------------- | ------------- | ------------- | ------------- |
| **Standard Hub & Spoke – NO bow-tie**  | NO | YES | NO | Not without peering the Azure regions |
| **Standard Hub & Spoke –bow-tie** | NO | YES | NO | Not without peering the Azure regions |
| **vWAN – NO bow-tie** | YES | YES | NO | YES |
| **vWAN – bow-tie** | YES | YES | NO | YES |

## ***Region failure***

| | **Can I still reach primary region?**  | **Can I reach secondary region?** | **Can I reach Azure without leveraging my MPLS?** | **Can I have connectivity between primary and secondary regions?** | 
| ------------- | ------------- | ------------- | ------------- | ------------- |
| **Standard Hub & Spoke – NO bow-tie**  | N.A | YES | NO | N.A |
| **Standard Hub & Spoke –bow-tie** | N.A | YES | YES | N.A |
| **vWAN – NO bow-tie** | N.A | YES | NO | N.A |
| **vWAN – bow-tie** | N.A | YES | YES | N.A |

# **Conclusions**

When we talk about “failures/outages” we often tend to forget about the importance of keeping in consideration **what** is exactly failing in the scenario we want to analyze.

The failure of a single link of an ExpressRoute circuit…

The failure of both links of an ExpressRoute circuit (so the failure of an entire peering location)…

The failure of ExpressRoute gateways inside a HUB (but not of the entire region)…

The failure of an entire Azure region…

All of these will produce very different effects depending on the analyzed scenario.

This premise is important as well when discussing the benefits of ExpressRoute’s cross-connections, which are of course introducing lots of added value in case of failure of Azure regions, but of course have to be considered useless if talking  about protection in case of failures on both the MSEE devices handling our circuit’s connectivity.

Considering the above summary tables, we could easily summarize our overall considerations with the following sentences:

1.	ExpressRoute’s cross-connections are indeed introducing added value to both standard Hub/Spoke and vWAN connectivity scenarios, but exclusively within the context of protecting against failures/outages which are not correlated with the availability of both the MSEE routers handling our circuit’s connectivity.
2.	vWAN (or vWAN-like connectivity topologies leveraging Hub/Spoke models) represents the real key-differentiator in protecting against both circuit’s and Azure region’s failures.

With vWAN we can still be able to preserve access to primary regions even in case of an ExpressRoute circuit’s failure, and with the bow-tie connections we can potentially be able to failover to a secondary region without impacting our MPLS connection. 


