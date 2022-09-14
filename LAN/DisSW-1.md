# DisSW-1 Configuration

## <ins>Initial Configuration</ins>

The naming convention of the hardware is [zone location]-\[architecture location][Hardware Type]-\[number].

- **Defining the hardware hostname**

    ios> `en`

    ios# `conf t`

    ios(config)# `hostname LAN-DisSW-1`


## <ins>Login Credentials</ins>

Even if it is not a production project, I like to get into the good habit of not exposing passwords and using the environment variables made available by Github.

- **Control access for enable (privileged) mode use of scrypt as the hashing algorithm**

    LAN-DisSW-1(config)# `enable algorithm-type secret scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Defining username/secret(password) for authentification**

    LAN-DisSW-1(config)# `username admin algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Configuring the command line to use locally configured username/secret pairs**

    LAN-DisSW-1(config)# `line con 0`

    LAN-DisSW-1(config-line)# `login local`

    LAN-DisSW-1(config-line)# `exit`

- **Configuring the terminal access to use locally configured username/secret pairs**

    LAN-DisSW-1(config)# `line vty 0 4`

    LAN-DisSW-1(config-line)# `login local`

    LAN-DisSW-1(config-line)# `exit`

## <ins>VLANs Configuration</ins>

As the Distribution Switch, it will needed to have all the VLAN presents in the Access Switch level. 
The VLAN 10 and 20 are for servers : Service (DCHP, DNS, NTP) and Backup.
The VLAN 30 and 40 are for endpoints : Dev and Office.
The VLAN 100 is choosen for management purpose (permit access to the configuration of the hardware components accross the LAN network).

-  **Configuring the VLANs name**

    LAN-DisSW-1(config)# `vlan 10`

    LAN-DisSW-1(config-vlan)# `name Service`

    LAN-DisSW-1(config-vlan)# `vlan 20`

    LAN-DisSW-1(config-vlan)# `name Backup`

    LAN-DisSW-1(config-vlan)# `vlan 30`

    LAN-DisSW-1(config-vlan)# `name Dev`

    LAN-DisSW-1(config-vlan)# `vlan 40`

    LAN-DisSW-1(config-vlan)# `name Office`

    LAN-DisSW-1(config-vlan)# `vlan 100`

    LAN-DisSW-1(config-vlan)# `name Management`

    LAN-DisSW-1(config-vlan)# `exit`

## <ins>Interface Configuration</ins>

- **Configuring the interfaces of the Distribution Switch for the connection to the Access Switch (use of the etherchannel to increase the bandwith)**

    LAN-DisSW-1(config)# `int range e0/0-1`

    LAN-DisSW-1(config-if-range)# `channel-group 1 mode active`

    LAN-DisSW-1(config-if-range)# `exit`

    LAN-DisSW-1(config)# `int port-channel 1`

    LAN-DisSW-1(config-if)# `Description Link to LAN-AccSW-1`

    LAN-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    LAN-DisSW-1(config-if)# `switchport mode trunk`

    LAN-DisSW-1(config-if)# `switchport trunk allowed vlan 10,20,100`

    LAN-DisSW-1(config-if)# `switchport nonegotiate`

    LAN-DisSW-1(config-if)# `exit`

    LAN-DisSW-1(config)# `int range e1/0-1`

    LAN-DisSW-1(config-if-range)# `channel-group 3 mode active`

    LAN-DisSW-1(config-if-range)# `exit`

    LAN-DisSW-1(config)# `int port-channel 3`

    LAN-DisSW-1(config-if)# `Description Link to LAN-AccSW-2`

    LAN-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    LAN-DisSW-1(config-if)# `switchport mode trunk`

    LAN-DisSW-1(config-if)# `switchport trunk allowed vlan 10,20,100`

    LAN-DisSW-1(config-if)# `switchport nonegotiate`

    LAN-DisSW-1(config-if)# `exit`

    LAN-DisSW-1(config)# `int range e2/0-1`

    LAN-DisSW-1(config-if-range)# `channel-group 5 mode active`

    LAN-DisSW-1(config-if-range)# `exit`

    LAN-DisSW-1(config)# `int port-channel 5`

    LAN-DisSW-1(config-if)# `Description Link to LAN-AccSW-3`

    LAN-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    LAN-DisSW-1(config-if)# `switchport mode trunk`

    LAN-DisSW-1(config-if)# `switchport trunk allowed vlan 30,40,100`

    LAN-DisSW-1(config-if)# `switchport nonegotiate`

    LAN-DisSW-1(config-if)# `exit`

    LAN-DisSW-1(config)# `int range e2/2-3`

    LAN-DisSW-1(config-if-range)# `channel-group 7 mode active`

    LAN-DisSW-1(config-if-range)# `exit`

    LAN-DisSW-1(config)# `int port-channel 7`

    LAN-DisSW-1(config-if)# `Description Link to LAN-AccSW-4`

    LAN-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    LAN-DisSW-1(config-if)# `switchport mode trunk`

    LAN-DisSW-1(config-if)# `switchport trunk allowed vlan 30,40,100`

    LAN-DisSW-1(config-if)# `switchport nonegotiate`

    LAN-DisSW-1(config-if)# `exit`

- **Configuring the interfaces of the Distribution Switch for the connection to the second Distribution Switch.**

    LAN-DisSW-1(config)# `int range e3/2-3`

    LAN-DisSW-1(config-if-range)# `channel-group 10 mode active`

    LAN-DisSW-1(config-if-range)# `exit`

    LAN-DisSW-1(config)# `int port-channel 10`

    LAN-DisSW-1(config-if)# `Description Link to LAN-DisSW-2`

    LAN-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    LAN-DisSW-1(config-if)# `switchport mode trunk`

    LAN-DisSW-1(config-if)# `switchport trunk allowed vlan 10,20,30,40,100`

    LAN-DisSW-1(config-if)# `switchport nonegotiate`

    LAN-DisSW-1(config-if)# `exit`

- **Switching off the unuse interface**

    LAN-DisSW-1(config)# `int range e0/2-3,e1/2-3`

    LAN-DisSW-1(config-if-range)# `shut`

    LAN-DisSW-1(config-if-range)# `exit`

## <ins>Inter-VLAN routing configuration</ins>

The Distribution Switch 1 will be prioritized for the routing of VLAN 10, 30 and 100. 

- **Configuring the VLAN interface for Inter-VLAN routing and HSRP**

    LAN-DisSW-1(config)# `int vlan 10`

    LAN-DisSW-1(config-if)# `ip add 172.16.10.252 255.255.255.0`

    LAN-DisSW-1(config-if)# `no shut`

    LAN-DisSW-1(config-if)# `standby 10 ip 172.16.10.254`

    LAN-DisSW-1(config-if)# `standby 10 priority 150`

    LAN-DisSW-1(config-if)# `standby 10 preempt`

    LAN-DisSW-1(config-if)# `exit`

    LAN-DisSW-1(config)# `int vlan 20`

    LAN-DisSW-1(config-if)# `ip add 172.16.20.252 255.255.255.0`

    LAN-DisSW-1(config-if)# `no shut`

    LAN-DisSW-1(config-if)# `standby 20 ip 172.16.20.254`

    LAN-DisSW-1(config-if)# `exit`

    LAN-DisSW-1(config)# `int vlan 30`

    LAN-DisSW-1(config-if)# `ip add 172.16.30.252 255.255.255.0`

    LAN-DisSW-1(config-if)# `no shut`

    LAN-DisSW-1(config-if)# `standby 30 ip 172.16.30.254`

    LAN-DisSW-1(config-if)# `standby 30 priority 150`

    LAN-DisSW-1(config-if)# `standby 30 preempt`

    LAN-DisSW-1(config-if)# `ip helper-address 172.16.10.11`

    LAN-DisSW-1(config-if)# `ip helper-address 172.16.10.12`

    LAN-DisSW-1(config-if)# `exit`

    LAN-DisSW-1(config)# `int vlan 40`

    LAN-DisSW-1(config-if)# `ip add 172.16.40.252 255.255.255.0`

    LAN-DisSW-1(config-if)# `no shut`

    LAN-DisSW-1(config-if)# `standby 40 ip 172.16.40.254`

    LAN-DisSW-1(config-if)# `ip helper-address 172.16.10.11`

    LAN-DisSW-1(config-if)# `ip helper-address 172.16.10.12`

    LAN-DisSW-1(config-if)# `exit` 

    LAN-DisSW-1(config)# `int vlan 100`

    LAN-DisSW-1(config-if)# `ip add 172.16.100.252 255.255.255.0`

    LAN-DisSW-1(config-if)# `no shut`

    LAN-DisSW-1(config-if)# `standby 100 ip 172.16.100.254`

    LAN-DisSW-1(config-if)# `standby 100 priority 150`

    LAN-DisSW-1(config-if)# `standby 100 preempt`

    LAN-DisSW-1(config-if)# `exit`    

- **Configuring the ACL for inter-VLAN Routing**

    For a better understanding of all the ACL rules implemented, take a look at the [flux matrix](Network_Flux_Matrix.md)

    LAN-DisSW-1(config)# `access-list 130 remark permit only access to the service's port of the server`

    LAN-DisSW-1(config)# `access-list 130 permit udp host 0.0.0.0 host 255.255.255.255 eq 67`

    LAN-DisSW-1(config)# `access-list 130 permit udp 172.16.30.0 0.0.0.255 host 172.16.10.11 eq 53`

    LAN-DisSW-1(config)# `access-list 130 permit udp 172.16.30.0 0.0.0.255 host 172.16.10.12 eq 53`

    LAN-DisSW-1(config)# `access-list 130 permit udp 172.16.30.0 0.0.0.255 host 172.16.20.11 eq 123`

    LAN-DisSW-1(config)# `access-list 130 permit udp 172.16.30.0 0.0.0.255 host 172.16.20.12 eq 123`

    LAN-DisSW-1(config)# `access-list 130 permit icmp 172.16.30.0 0.0.0.255 172.16.10.0 0.0.255`

    LAN-DisSW-1(config)# `access-list 130 permit icmp 172.16.30.0 0.0.0.255 172.16.20.0 0.0.255`    

    LAN-DisSW-1(config)# `access-list 130 deny ip 172.16.30.0 0.0.0.255 172.16.10.0 0.0.0.255`

    LAN-DisSW-1(config)# `access-list 130 deny ip 172.16.30.0 0.0.0.255 172.16.40.0 0.0.0.255`   

    LAN-DisSW-1(config)# `access-list 130 permit ip 172.16.30.0 any`

    LAN-DisSW-1(config)# `int vlan 30`

    LAN-DisSW-1(config)# `ip access-group 130 in`

    LAN-DisSW-1(config)# `exit`

    LAN-DisSW-1(config)# `access-list 140 remark permit only access to the service's port of the server`

    LAN-DisSW-1(config)# `access-list 140 permit udp host 0.0.0.0 host 255.255.255.255 eq 67`

    LAN-DisSW-1(config)# `access-list 140 permit udp 172.16.40.0 0.0.0.255 host 172.16.10.11 eq 53`

    LAN-DisSW-1(config)# `access-list 140 permit udp 172.16.40.0 0.0.0.255 host 172.16.10.12 eq 53`

    LAN-DisSW-1(config)# `access-list 140 permit udp 172.16.40.0 0.0.0.255 host 172.16.20.11 eq 123`

    LAN-DisSW-1(config)# `access-list 140 permit udp 172.16.40.0 0.0.0.255 host 172.16.20.12 eq 123`

    LAN-DisSW-1(config)# `access-list 140 permit icmp 172.16.40.0 0.0.0.255 172.16.10.0 0.0.255`

    LAN-DisSW-1(config)# `access-list 140 permit icmp 172.16.40.0 0.0.0.255 172.16.20.0 0.0.255`    

    LAN-DisSW-1(config)# `access-list 140 deny ip 172.16.40.0 0.0.0.255 172.16.10.0 0.0.0.255`

    LAN-DisSW-1(config)# `access-list 140 deny ip 172.16.40.0 0.0.0.255 172.16.30.0 0.0.0.255`   

    LAN-DisSW-1(config)# `access-list 140 permit ip 172.16.40.0 any`

    LAN-DisSW-1(config)# `int vlan 40`

    LAN-DisSW-1(config)# `ip access-group 140 in`

    LAN-DisSW-1(config)# `exit`
    
## <ins>Spanning-Tree Configuration</ins>

- **Configuring the Spanning-Tree priority for choosing the root switch**

    The same choice for the switch priority have been taken for spanning-tree and inter-vlan routing.

    LAN-DisSW-1(config)# `spanning-tree vlan 10 root primary`

    LAN-DisSW-1(config)# `spanning-tree vlan 20 root secondary`

    LAN-DisSW-1(config)# `spanning-tree vlan 30 root primary`

    LAN-DisSW-1(config)# `spanning-tree vlan 40 root secondary`
    
    LAN-DisSW-1(config)# `spanning-tree vlan 100 root primary`


