# DHCP Configuration
---
The DHCP service is already installed on the Docker image (debian-server). 
The server will have as an IP address : 172.16.10.1/24 (gateway : 172.16.10.254)

## Listening interface

The listening interface of the DHCP Server must be first indicated in the file `/etc/default/isc-dchp-server`.

Code BASH :

    sudo vim /etc/default/isc-dhcp-server

The listening interface, in our case, is only for IPv4 :

    INTERFACESv4="eth0"

## Lease configuration

If the server has access to more than one subnet, DHCP requires all subnet to be defined even though there isn't intention to enable DHCP service on that subnet.
So the servers' subnet will have a empty subnet.

Code BASH :

    sudo vim /etc/dhcp/dhcp.conf

In the file, adding the following line for leasing subnet for the computers in VLAN 30 and 40 :

    default-lease-time 3600;
    max-lease-time 7200;
    
    # Network Servers
    subnet 172.16.10.0 netmask 255.255.255.0 {
    }

    # Network Dev
    subnet 172.16.30.0 netmask 255.255.255.0 {
        option broadcast-address    172.16.30.255;
        range                       172.16.30.50 172.16.30.250;
        option domain-name-servers  172.16.10.11, 172.16.10.12;
        option routers              172.16.30.254;
    }

    # Network Office
    subnet 172.16.40.0 netmask 255.255.255.0 {
        option broadcast-address    172.16.40.255;
        range                       172.16.40.50 172.16.40.250;
        option domain-name-servers  172.16.10.11, 172.16.10.12;
        option routers              172.16.40.254;
    }

## Start DHCP service

Code BASH :

    sudo service isc-dhcp-server start


## Client configuration

To configure the computer in VLAN 30 and 40 to use DHCP on a network interface eth0 for Debian systems.

Code BASH :

    sudo vim /etc/network/interfaces

Enter the following lines in this file :

    auto eth0
    iface eth0 inet DHCP

## Information

As the DCHP server is not in the same subnet that the computer, a relay is needed. In our case, it will be the Distribution switch on the interface vlan (30 and 40) with the use of `ip helper-address` command to forward the broadcast packets (DCHP discover and DHCP request) at destination of the DHCP Server :

    LAN-DisSW-1(config-if)# ip helper-address 172.16.10.11
