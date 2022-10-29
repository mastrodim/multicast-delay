
In this experiment, we will see how multicast routing protocols can be used to mitigate delay in blockage-prone mmWave networks. It should take about 90 minutes to run this experiment.

This experiment runs on the CloudLab testbed. To reproduce this experiment on Cloudlab, you will need an account on [Cloudlab](href="https://cloudlab.us/), you will need to have [joined a project](https://docs.cloudlab.us/users.html#%28part._join-project%29), and you will need to have [set up SSH access](https://docs.cloudlab.us/users.html#%28part._ssh-access%29).


## Background



Redundancy - sending multiple copies of data across different network paths - has the potential to mitigate poor reliability and delay performance in mesh networks. However, this has not been fully explored because mesh networks were traditionally subject to tight capacity constraints that made redundant transmissions less practical. With the recent availability of mmWave links that have very high capacity but poor reliability, the potential of this approach should be revisited. If it can deliver improved reliability and delay performance on high-capacity mmWave links, it can enable new applications like remote surgery and cloud-controlled autonomous driving. To address this, we develop a protocol for one-to-one data delivery with redundancy using multicast protocols, and evaluate it in a testbed environment that is representative of a mmWave mesh network. The results of this research will inform further protocol design and development for reliable low-latency communication over mmWave links.


## Result

The following CFD plot shows how redundancy mitigates the poor reliability and reduces the delay. The graph shows that introducing more links and thus redundancy results in huge drops in the delay. The more links that are introduced, the majority of the data packets experience less delay.

![Untitled drawing(2)](https://user-images.githubusercontent.com/57250247/198820399-c8f01d0b-550b-43df-b36c-f1188ad25efc.png)


## Run my experiment

To produce the results above, we ran a sequence of experiments on CloudLab to explore how multicasting can mitigate the delay created due to the poor reliability of mmWave links. In this profile, we provide instructions for running these experiments, to reproduce our results.

## Reserve Resources

To run these experiments, you will need an account on CloudLab. If you already have a GENI account, you can use it to log in to CloudLab. Once you are logged in to CloudLab, open our multicast-full-v1 profile: 

https://www.cloudlab.us/instantiate.php?profile=c21c760e-09d7-11ed-b318-e4434b2381fc&from=manage-profile

Click on "Instantiate", then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the "List view" tab to get SSH login instructions for your nodes. Then, use SSH to open a shell session on each node in the experiment. 

This should be the topology in front of you:


![MastrogiannisDimitrios_poster_2022](https://user-images.githubusercontent.com/57250247/185206033-2ad85c6c-3759-450d-9ad1-a44686d6c992.jpg)

The role of each node in the topology is explained as follows:

* rp will be the rendezvous point, the root of the shared tree in PIM-SM "Any-Source Multicast" mode.
* the routers cr1 and cr2 represent core routers.
* the router fhr1 is directly connected to source1. Routers that are directly connected to a multicast source are known as first-hop routers. The links between source1, fhr1,rp, cr1 and cr2 are gigabit capacity fiber links.
* the routers lhr1, lhr2, lhr3, and lhr4 are directly connected to the interfaces of the host that will be the multicast receiver: rx. Routers that are directly connected to a multicast receiver are known as last-hop routers.


### Configure Routing 

Configure routing on source:
On source 1 run:

```
sudo route add -net 10.10.0.0/16 gw 10.10.101.1
sudo route add -net 224.0.0.0 netmask 240.0.0.0 dev eth1
```

#### Configure unicast routing 

In all of the routers, open the router configuration terminal with:
```
VTYSH_PAGER=more
sudo vtysh
```
At the FRR shell, run
```
show ip route
```
To configure OSPF, enter Global Configuration mode in each router:
```
configure terminal
``` 

Then, type
```
router ospf
```  

to enable OSPF


Finally, you need to associate one or more networks to the OSPF routing process. Run

```
network 10.10.0.0/16 area 0.0.0.0
``` 

so that all addresses from 10.10.0.0-10.10.255.255 will be enabled for OSPF.

#### Configure multicast routing

Once the unicast routing protocol is set up, we can configure multicast routing.
First, we will prepare the rendezvous point. At the FRR shell on the rp router, run:
```
configure terminal  
int eth1  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4  
ip pim spt-switchover infinity  
exit
```
Next, we will configure the two core routers, cr1 and cr2. On these routers, we will turn on PIM-SM and set the RP address for all multicast groups to 10.10.1.100, as with the RP. However, this router has several network interfaces, so we will need to repeat these steps for each interface.

At the FRR shell on cr1 and cr2, run:
```
configure terminal

int eth1  
ip pim sm
ip pim rp 10.10.1.100 224.0.0.0/4

int eth2  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4

int eth3  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4

ip pim spt-switchover infinity

exit  
```
Then, we will configure the router connected to the multicast source: fhr1. At the FRR shell on fhr1, run:
```
configure terminal

int eth1  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4

int eth2  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4

ip pim spt-switchover infinity

exit  
```
Finally, we will configure the routers connected to the multicast receivers: lhr1, lhr2, lhr3, lhr4. On these routers, we will also need to enable IGMP, since these routers will use the IGMP Join messages from receivers to build the multicast tree. At the FRR shell on these routers, run:

```
configure terminal

int eth1  
ip igmp  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4

int eth2  
ip igmp  
ip pim sm  
ip pim rp 10.10.1.100 224.0.0.0/4

ip pim spt-switchover infinity

exit
```

#### Router Configuration

At this point, we will configure the interface on the routers through which the traffic goes to the destination node. We will use tc to set up an HTB queue with an egress rate of 1Gbps. We will add a FIFO queue with a limit of 100MB. The maximum size of 4K quality video packets is 20MB per second. Thus, we added a big enough limit in order not to have packet drops.

For each router, find the address of the interface through which traffic flows to the destination and run the following commands:
```
sudo tc qdisc del dev $(ip route get 10.10.105.2 | grep -oP "(?<=dev )[^ ]+") root  

sudo tc qdisc replace dev $(ip route get 10.10.105.2 | grep -oP "(?<=dev )[^ ]+") root handle 1: htb default 3  

sudo tc class add dev $(ip route get 10.10.105.2 | grep -oP "(?<=dev )[^ ]+") parent 1: classid 1:3 htb rate 1gbit

sudo tc qdisc add dev $(ip route get 10.10.105.2 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 100000000

```

In the example above, the interface had an address of 10.10.105.2.

The address of each interface can be found in the CloudLab network topology view by clicking on the respective network element.



### Applying Random Blockages

At this point, we will implement random blockages on certain links that reflect the behavior of mmWave links. Those blockages will be implemented to mimic the behavior of mmWave links which are prone to experience blockages. In our experiment, we will suppose that only the links after the routers cr1 cr2 are mmWave links and that the ones before them are fiber links that experience negligible blockages.

So, we will implement the blockages in all the interfaces of routers cr1, cr2, lhr1, lhr2, lhr3, lhr4.

We will create a filename.sh gist in GitHub with the following code:

![Blockage](https://user-images.githubusercontent.com/57250247/187446617-61651084-245b-49eb-84a3-26daa192f83b.png)

At this point we should download each file to the respective receiver. So, we will open a router window for each interface and type:
```
wget link
```
where "link" is the URL link to the respective gist file you created.

Then, in order to run the code, open a router window for each interface and type:
```
bash blockage.sh
```
where "blockage" is the name of the respective gist file.

### Enabling the interfaces of the receiver

At this stage, we will enable the 4 different interfaces of the receiver(rx) to receive traffic through them and thus achieve redundancy.
```
sudo ip addr add 239.255.12.42 dev eth1 autojoin
sudo ip addr add 239.255.12.42 dev eth2 autojoin 
sudo ip addr add 239.255.12.42 dev eth3 autojoin
sudo ip addr add 239.255.12.42 dev eth4 autojoin
```
### Synchronizing the network elements

As the experiment revolves around calculating delay, it is important to have all the network elements in sync.
In order to sync them, we will employ NTP.
We will run the following commands in the rx window and source1 window.
```
sudo service ntp stop
sudo ntpd -gq -d 1 0.us.pool.ntp.org 1.us.pool.ntp.org
sudo service ntp start
```
### Capturing and saving the data packets

Now, we will move on to capture the data packets that are send from the sender (source1) to the receiver through its 4 interfaces.

In a sender(source 1) window, type:
```
sudo tcpdump -i eth1 -s 96 -w rec-send-ntp-tcpdump-snaplen.pcap
```
Create four receiver(rx) windows for each of the four interfaces.

In the "eth1" receiver window run:
```
sudo tcpdump -i eth1 -s 96 -w rec-eth1-ntp-tcpdump-snaplen.pcap
```
In the "eth2" receiver window run:
```
sudo tcpdump -i eth2 -s 96 -w rec-eth2-ntp-tcpdump-snaplen.pcap
```
In the "eth3" receiver window run:
```
sudo tcpdump -i eth3 -s 96 -w rec-eth3-ntp-tcpdump-snaplen.pcap
```
In the "eth4" receiver window run:
```
sudo tcpdump -i eth4 -s 96 -w rec-eth4-ntp-tcpdump-snaplen.pcap
```
### Enabling the Video Stream

In a sender (source1) window, we will run the following command to start a video stream:
```
vlc --intf ncurses -aout dummy hdvideo.mp4 --sout udp:239.255.12.42 --ttl 6 --repeat
```
In a receiver (rx) window run:
```
vlc --intf ncurses --vout dummy --aout dummy udp://@239.255.12.42 --repeat
```

The above enables the video stream traffic to reach the receiver as the receiver joins the multicast tree.


### Stopping the experiment

After 3.5 minutes have elapsed, stop the video stream at the sender(source1) and receiver(rx) window by typing Ctrl+C in each window.
We choose to stop the video stream after 3.5 minutes in order to avoid a loop in the video stream playback that could mix the data collected.
Then, in each of the windows where data is captured also stop the capturing by typing Ctrl+C.


### Converting the .pcap file to a .csv file

We will open a new sender (source1) window and type the following code:
```
tshark -r rec-send-ntp-tcpdump-snaplen.pcap  -n  -Y  "udp.port==1234" -T fields -e frame.time_epoch -e ip.id  -e frame.len -e ip.src -e ip.dst -e eth.addr -E separator=, >rec-send-ntp-tcpdump-snaplen.csv
```
We will open a new receiver (rx) window and type the following code:
```
tshark -r rec-eth1-ntp-tcpdump-snaplen.pcap  -n  -Y  "udp.port==1234" -T fields -e frame.time_epoch -e ip.id  -e frame.len -e ip.src -e ip.dst -e eth.addr -E separator=, >rec-eth1-ntp-tcpdump-snaplen.csv

tshark -r rec-eth2-ntp-tcpdump-snaplen.pcap  -n  -Y  "udp.port==1234" -T fields -e frame.time_epoch -e ip.id  -e frame.len -e ip.src -e ip.dst -e eth.addr -E separator=, >rec-eth2-ntp-tcpdump-snaplen.csv

tshark -r rec-eth3-ntp-tcpdump-snaplen.pcap  -n  -Y  "udp.port==1234" -T fields -e frame.time_epoch -e ip.id  -e frame.len -e ip.src -e ip.dst -e eth.addr -E separator=, >rec-eth3-ntp-tcpdump-snaplen.csv

tshark -r rec-eth4-ntp-tcpdump-snaplen.pcap  -n  -Y  "udp.port==1234" -T fields -e frame.time_epoch -e ip.id  -e frame.len -e ip.src -e ip.dst -e eth.addr -E separator=, >rec-eth4-ntp-tcpdump-snaplen.csv

```
### Data Analysis


The data analysis is conducted in a Python Notebook, that can be found in the following link:

https://colab.research.google.com/drive/1rYC-nQqZDDdQ_XEzRBsgyTOvidaYZisX?usp=sharing


There, the .csv files can be uploaded and by running the notebook commands, the data will be analyzed and the results will be presented in a graphic format.

## Notes

A video explaining the research project can be found in the following link: https://www.youtube.com/watch?v=VuEi0e0Ybbk

## References

1. A. Koutsaftis, M. Ozkoc, F. Fund, P. Liu and S. S. Panwar, "Fast Wireless Backhaul: A Multi-Connectivity Enabled mmWave Cellular System," to appear in  2022 IEEE Global Communications Conference: Communication QoS, Reliability and Modeling, 2022.

2. A. Srivastava, F. Fund and S. S. Panwar, "An Experimental Evaluation of Low Latency Congestion Control for mmWave Links," IEEE INFOCOM 2020 - IEEE Conference on Computer Communications Workshops (INFOCOM WKSHPS), 2020, pp. 352-357, doi: 10.1109/INFOCOMWKSHPS50562.2020.9162881.

3. F. Fund, “NTP exercises,” Internet Architecture and Protocols. [Online]. Available: https://ffund.github.io/tcp-ip-essentials/lab8/el5373-lab8-89. [Accessed: 30-Aug-2022]. 

4. F. Fund, “Multicast routing with PIM,” Run my testbed experiment, 25-Apr-2022. [Online]. Available: https://witestlab.poly.edu/blog/multicast-routing-with-pim/. [Accessed: 30-Aug-2022]. 

5. A. Srivastava, “An experimental evaluation of low latency congestion control over mmwave links,” Run my testbed experiment, 31-Mar-2020. [Online]. Available: https://witestlab.poly.edu/blog/tcp-mmwave/. [Accessed: 30-Aug-2022]. 

6. F. Fund, “1.5 using tcpdump and Wireshark,” Internet Architecture and Protocols. [Online]. Available: https://ffund.github.io/tcp-ip-essentials/lab1/1-5-tcpdump-wireshark.html. [Accessed: 30-Aug-2022]. 
