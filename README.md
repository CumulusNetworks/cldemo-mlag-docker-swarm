# DEPRECATED
## This repo is no longer maintained.<br>For a list of current demos, please visit:<br>https://gitlab.com/cumulus-consulting/goldenturtle/<br><br><br>

This demo shows one of several different approaches to running Docker. This approach uses Docker Swarm to create overlay (vxlan) tunnels between the servers.   Extra redundancy to the hosts is provided using Cumulus MLAG between the leafs and the servers.  BGP unnumbered is configured between the leafs and the spines. An out-of-band management network is also configured.  More information on this setup can be found in the Cumulus Networks Design Guide: [Container Networking with Cumulus Linux Validated Design Guide: Using Docker Swarm with MLAG or Single Attached Hosts](https://cumulusnetworks.com/learn/web-scale-networking-resources/validated-design-guides/Container-with-VXLAN-and-Docker-Swarm-with-MLAG-or-single-attached-to-the-hosts/)


The virtual setup is depicted below:
![Virtual  Demo Topology](https://github.com/CumulusNetworks/cldemo-mlag-docker-swarm/blob/master/cldemo-mlag-docker-swarm.png)

A docker swarm management node [server01] is configured and 3 additional worker nodes [server02, server03, and server04] are configured. Server02 and Server03 are promoted to management nodes for redundancy.  All management nodes are also a worker nodes. 

When a worker node joins a swarm, Docker Swarm creates VXLAN tunnels between the worker node and the other worker and management node[s] for inter-container, inter-node communication using the overlay driver.  The VTEPs are on the servers. In this demo, the VTEPs are configured to be the server mlag bond IP addresses.  Docker Swarm also uses the bridge driver on a network called docker_gwbridge to access the containers from outside the vxlan.  

The management node [Server01] then creates an apache service [ 3 instances of apache containers] on the worker nodes (which may include server01). (If more are required, edit the /group_vars/all file services.replicas value in the playbook after you have cloned it locally)

We can access the replicated apache containers from either outside the VXLAN via CURL on port 8080, and/or access within the VXLAN (container to container) via ping. 

Software in Use:
----------------

On Spines and Leafs:

 - Cumulus v3.3.2

On Servers:

 - Ubuntu 16.04 
 - Docker-CE v17.05 
 - apache (container image)
  

Quickstart: Run the demo
------------------------

Before running this demo, install VirtualBox and Vagrant. The currently supported versions of VirtualBox and Vagrant can be found in the [cldemo-vagrant](https://github.com/CumulusNetworks/cldemo-vagrant) prequisites section.
 

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch
    vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02
    vagrant up server01 server02 server03 server04
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    git clone https://github.com/cumulusnetworks/cldemo-mlag-docker-swarm
    cd cldemo-mlag-docker-swarm
    ansible-playbook run-demo.yml
    ssh server01
    
    
Check the Docker Swarm Setup:

    cumulus@server01:~$ sudo docker node ls
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
    5j4ur5z70jhg18tibmti59eqi     server03            Ready               Active              Reachable
    cvaw88qajb8ovn0ip22adaeo8     server04            Ready               Active              
    naptasyz6qjqd3292y6lvbrso *   server01            Ready               Active              Leader
    uoq6z23klm0l4i5387nv8p3ul     server02            Ready               Active              Reachable

    


View the Services:

    cumulus@server01:~$ sudo docker service ls
    ID                  NAME                MODE                REPLICAS            IMAGE               PORTS 
    zajgxoinvvb9        apache_web          replicated          3/3                 php:5.6-apache      *:8080->80/tcp
    

Curl to the service from a leaf node using port 8080 [10.0.0.32 is loopback of server02] from the default VRF:

    cumulus@leaf01:mgmt-vrf:~$ sudo vrf task exec default curl 10.0.0.32:8080
    

Check Container Interfaces:

    cumulus@server02:~$ sudo docker ps -a
    cumulus@server02:~$ sudo docker exec -it <container id> /bin/bash
    root@0fa93f199e9e:/var/www/html# ip addr show
    root@0fa93f199e9e:ping <ip of overlay on remote serve>
    




