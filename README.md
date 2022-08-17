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


## Set Up Resources 

This section describes how to prepare your nodes to run this experiment:


## Notes



### References
