In this experiment, we will see how multicast routing protocols can be used to mitigate delay in blockage-prone mmWave networks. It should take about 90 minutes to run this experiment.

This experiment runs on the CloudLab testbed. To reproduce this experiment on Cloudlab, you will need an account on [Cloudlab](href="https://cloudlab.us/), you will need to have [joined a project](https://docs.cloudlab.us/users.html#%28part._join-project%29), and you will need to have [set up SSH access](https://docs.cloudlab.us/users.html#%28part._ssh-access%29).


## Background


Redundancy - sending multiple copies of data across different network paths - has the potential to mitigate poor reliability and delay performance in mesh networks. However, this has not been fully explored because mesh networks were traditionally subject to tight capacity constraints that made redundant transmissions less practical. With the recent availability of mmWave links that have very high capacity but poor reliability, the potential of this approach should be revisited. If it can deliver improved reliability and delay performance on high-capacity mmWave links, it can enable new applications like remote surgery and cloud-controlled autonomous driving. To address this, we develop a protocol for one-to-one data delivery with redundancy using multicast protocols, and evaluate it in a testbed environment that is representative of a mmWave mesh network. The results of this research will inform further protocol design and development for reliable low-latency communication over mmWave links.


## Result

The following CFD plot shows how redundancy mitigates the poor reliability and reduces the delay. The graph shows that introducing more links and thus redundancy results in huge drops in the delay. The more links that are introduced, the majority of the data packets experience less delay.

![PICTURE2](https://user-images.githubusercontent.com/57250247/185035728-40b739e1-0254-4787-8319-691ff1212c24.jpg)




## Run my experiment

To produce the results above, we ran a sequence of experiments on CloudLab to explore how multicasting can mitigate the delay created due to the poor reliability of mmWave links. In this profile, we provide instructions for running these experiments, in order to reproduce our results.

## Reserve Resources

To run these experiments, you will need an account on CloudLab. If you already have a GENI account, you can use it to log in to CloudLab. Once you are logged in to CloudLab, open our multicast-full-v1 profile: 

https://www.cloudlab.us/instantiate.php?profile=c21c760e-09d7-11ed-b318-e4434b2381fc&from=manage-profile

Click on "Instantiate", then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the "List view" tab to get SSH login instructions for your nodes. Then, use SSH to open a shell session on each node in the experiment. 

This should be the topology in front of you:


![MastrogiannisDimitrios_poster_2022](https://user-images.githubusercontent.com/57250247/185206033-2ad85c6c-3759-450d-9ad1-a44686d6c992.jpg)

The role of each node in the topology is explained as follows:

* rp will be the rendezvous point, the root of the shared tree in PIM-SM "Any-Source Multicast" mode.
* the routers cr1 and cr2 represent core routers.
* the router fhr1 and fhr2 is directly connected to source1. Routers that are directly connected to a multicast source are known as first hop routers. The links between source1, fhr1,rp, cr1 and cr2 are gigabit capacity fiber links.
* the routers lhr1, lhr2, lhr3, lhr4 are directly connected to the interfaces of the host that will be the multicast receiver: rx .Routers that are directly connected to a multicast receiver are known as last hop routers.


## Set Up Resources 

This section describes how to prepare your nodes to run this experiment:

##Configure routing on source:
On source 1 run:

`sudo route add -net 10.10.0.0/16 gw 10.10.101.1`
`sudo route add -net 224.0.0.0 netmask 240.0.0.0 dev eth1`

##Configure unicast routing 

In all of the routers, open the router configuration terminal with:

`VTYSH_PAGER=more`
`sudo vtysh`

At the FRR shell, run

`show ip route`

To configure OSPF, enter Global Configuration mode in each router:

`configure terminal`  

Then, type

`router ospf`  

to enable OSPF


Finally, you need to associate one or more networks to the OSPF routing process. Run

`network 10.10.0.0/16 area 0.0.0.0`  

so that all addresses from 10.10.0.0-10.10.255.255 will be enabled for OSPF.

##Configure multicast routing

Once the unicast routing protocol is set up, we can configure multicast routing.
First, we will prepare the rendezvous point. At the FRR shell on the rp router, run:

`configure terminal`  
`int eth1`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`  
`ip pim spt-switchover infinity`  
`exit`  

Next, we will configure the two core routers, cr1 and cr2. On these routers, we will turn on PIM-SM and set the RP address for all multicast groups to 10.10.1.100, as with the RP. However, this router has several network interfaces, so we will need to repeat these steps for each interface.

At the FRR shell on cr1 and cr2, run:

`configure terminal`

`int eth1`  
`ip pim sm`
`ip pim rp 10.10.1.100 224.0.0.0/4`


`int eth2`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`


`int eth3`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`

`ip pim spt-switchover infinity`

`exit`  

Then, we will configure the router connected to the multicast source: fhr1. At the FRR shell on fhr1, run:

`configure terminal`

`int eth1`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`


`int eth2`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`

`ip pim spt-switchover infinity`

`exit`  

Finally, we will configure the routers connected to the multicast receivers.: lhr1, lhr2, lhr3, lhr4 On these routers, we will also need to enable IGMP, since these routers will use the IGMP Join messages from receivers to build the multicast tree. At the FRR shell on these routers, run:

`configure terminal`

`int eth1`  
`ip igmp`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`


`int eth2`  
`ip igmp`  
`ip pim sm`  
`ip pim rp 10.10.1.100 224.0.0.0/4`

`ip pim spt-switchover infinity`

`exit` 

##Apply Random Blockages

At this point, we will implement random blockages on certain links that reflect the behavior of mmWave links.

## Notes



### References
