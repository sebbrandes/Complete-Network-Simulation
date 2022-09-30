Pacemaker uses Corosync for heartbeat and internal communication among cluster components. 
Pacemaker manages all cluster resources and achieves maximum availability by detecting and recovering from node level failures 
by making use of the messaging and membership capabilities provided by Corosync.

In the present case, the Docker images are based on Debian 11 (Bullseye).
The second layer is a base layer of all images used in the context of the simulation. It is composed of different network tools.
The third layer includes all the elements (Nginx, Pacemaker, Corosync and crmsh) necessary for setting up the load balancer cluster.

The following Ip addresses are used :

    172.16.80.141/25 	cluster Virtual IP

    172.16.80.131/25 	nginx1 (IP outside)
    172.16.80.132/25 	nginx2 (IP outside)

    172.16.80.11/25		nginx1 (IP inside)
    172.16.80.12/25		nginx2 (IP inside)

    172.16.80.1/25 		webserver1
    172.16.80.2/25		webserver2
    172.16.80.3/25	    webserver3
    172.16.80.4/25		webserver4

## Configuring Corosync

On both cluster's nodes (nginx1|nginx2) :

    nano /etc/hosts

In the file :

    172.16.80.131	nginx1
    172.16.80.132	nginx2

### Generate the Corosync key for the cluster authentification

On nginx1:
    
    corosync-keygen

Transfer the generate key to the second node in our cluster :

    scp /etc/corosync/authkey root@nginx2:/etc/corosync

### Enable the start of the Corosync service

On both cluster's nodes (nginx1|nginx2) :

    nano /etc/default/corosync

In the file :

    START=yes

### Cluster configuration

All explanations in the configuration file is coming from the official documentation but only the parts concerning the deployed configuration have been incorporated.

On both cluster's nodes (nginx1|nginx2) :

    mv /etc/corosync/corosync.conf /etc/corosync/corosync-default.conf
    nano /etc/corosync/corosync.conf

In the file :

    #ringnumber
    #This specifies the ring number for the interface. 
    #When using the redundant ring protocol, each interface should specify separate ring numbers 
    #to uniquely identify to the membership protocol which interface to use for which redundant ring. 
    #The ringnumber must start at 0.

    #bindnetaddr
    #This specifies the network address the corosync executive should bind to.

    totem {
        version: 2
        cluster_name: nginx
        interface {
            ringnumber: 0
            bindnetaddr: 172.16.80.128
            mcastport: 5405
        }
    }

    #votequorum reads its configuration from corosync.conf. 
    #Some values can be changed at runtime, others are only read at corosync startup. 
    #It is very important that those values are consistent across 
    #all the nodes participating in the cluster or votequorum behavior will be unpredictable.

    #votequorum requires an expected_votes value to function, this can be provided in two ways. 
    #The number of expected votes will be automatically calculated when the nodelist { } section is present 
    #in corosync.conf or expected_votes can be specified in the quorum { } section. 
    #Lack of both will disable votequorum. 
    #If both are present at the same time, the quorum.expected_votes value will override the one 
    #calculated from the nodelist. 

    #The "two node cluster" is a use case that requires special consideration. 
    #With a standard two node cluster, each node with a single vote, there are 2 votes in the cluster. 
    #Using the simple majority calculation (50% of the votes + 1) to calculate quorum, the quorum would be 2. 
    #This means that the both nodes would always have to be alive for the cluster to be quorate and operate.
    #Enabling two_node: 1, quorum is set artificially to 1. 

    quorum {
        provider: corosync_votequorum
        two_nodes: 1
    }

    nodelist {
        node {
            ring0_addr: 172.16.80.131
            name: primary
            nodeid: 1
        }
        node {
            ring0_addr: 172.16.80.132
            name: secondary
            nodeid: 2
        }
    }

    logging {
        to_logfile: yes
        to_syslog: yes
        logfile: /var/log/corosync/corosync.log
        timestamp: on
    }

    service {
        name: pacemaker
        ver: 1
    }

### Starting services

On both cluster's nodes (nginx1|nginx2):

    /etc/init.d/corosync start
    /etc/init.d/pacemaker start

The following command allow to check the Corosync members.

    corosync-cmapctl | grep members

And the result :

    runtime.members.1.config_version (u64) = 0
    runtime.members.1.ip (str) = r(0) ip(172.16.80.131)
    runtime.members.1.join_count (u32) = 1
    runtime.members.1.status (str) = joined
    runtime.members.2.config_version (u64) = 0
    runtime.members.2.ip (str) = r(0) ip(172.16.80.132)
    runtime.members.2.join_count (u32) = 1
    runtime.members.2.status (str) = joined

### Configuring the Cluster

The Pacemaker commands will be run on Primary Node (nginx1), as it automatically synchronizes all cluster-related changes across all member nodes.

The STONITH mode will be disable : STONITH is a mode that can be used to remove faulty nodes but like it is a two node cluster, the STONITH mode is not required.

It will be disabled with the following command :

crm configure property stonith-enabled=false

A virtual IP resource is created for the floating IP using the crm command (package crmsh) :

    crm configure primitive virtual_ip ocf:heartbeat:IPaddr2 params ip="172.16.80.133" cidr_netmask="32" op monitor interval="10s" meta migration-threshold="10"

### Checking the installation

Display the cluster status on the console just once then exit: 

    crm_mon -1

And the result :

    Cluster Summary:
    * Stack: corosync
    * Current DC: primary (version 2.0.5-ba59be7122) - partition with quorum
    * 2 nodes configured
    * 1 resource instance configured

    Node List:
    * Online: [ primary secondary ]

    Active Resources:
    * virtual_ip (ocf:heartbeat:IPaddr):		Started primary