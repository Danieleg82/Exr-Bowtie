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
