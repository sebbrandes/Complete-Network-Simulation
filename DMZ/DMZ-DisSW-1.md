# DMZ-DisSW-1 configuration

## <ins>Initial Configuration</ins>

The naming convention of the hardware is [zone location]-\[architecture location][Hardware Type]-\[number].

- **Define the hardware hostname**

    ios> `en`

    ios# `conf t`

    ios(config)# `hostname DMZ-DisSW-1`

## <ins>Login Credentials</ins>

Even if it is not a production project, I like to get into the good habit of not exposing passwords and using the environment variables made available by Github.

- **Control access for enable (privileged) mode use of scrypt as the hashing algorithm**

    DMZ-DisSW-1(config)# `enable algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Define username/secret(password) for authentification**

    DMZ-DisSW-1(config)# `username admin algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Commmand line configuration to use locally configured username/secret pairs**

    DMZ-DisSW-1(config)# `line con 0`

    DMZ-DisSW-1(config-line)# `login local`

    DMZ-DisSW-1(config-line)# `exit`

- **Terminal access configuration to use locally configured username/secret pairs**

    DMZ-DisSW-1(config)# `line vty 0 4`

    DMZ-DisSW-1(config-line)# `login local`

    DMZ-DisSW-1(config-line)# `exit`

## <ins>VLANs Configuration</ins>

This switch is connected to the different servers.
VLAN 50 for the communication between the Switch and the Firewalls 
VLAN 70 for the forwarding Proxy and DNS public resolution. 
VLAN 80 for the reverse Proxy/Load Balancer and Web servers.
And finally, VLAN 100 for management purpose.

- **Define the VLANs name**

    DMZ-DisSW-1(config)# `vlan 50`

    DMZ-DisSW-1(config-vlan)# `name Network`

    DMZ-DisSW-1(config)# `vlan 70`

    DMZ-DisSW-1(config-vlan)# `name Internet`

    DMZ-DisSW-1(config-vlan)# `vlan 80`

    DMZ-DisSW-1(config-vlan)# `name Public`

    DMZ-DisSW-1(config-vlan)# `vlan 100`

    DMZ-DisSW-1(config-vlan)# `name Management`

    DMZ-DisSW-1(config-vlan)# `exit`

## <ins> Interface Configuration</ins>

- **Configuration of the Distribution Switch interfaces for the connection to the Access switch (use of etherchannel to increase the bandwith)**

    DMZ-DisSW-1(config)# `int range e1/0-1`

    DMZ-DisSW-1(config-if-range)# `channel-group 21 mode active`

    DMZ-DisSW-1(config-if-range)# `exit`

    DMZ-DisSW-1(config)# `int port-channel 21`

    DMZ-DisSW-1(config-if)# `Description Link to DMZ-AccSW-1`

    DMZ-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-DisSW-1(config-if)# `switchport mode trunk`

    DMZ-DisSW-1(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-DisSW-1(config-if)# `switchport nonegotiate`

    DMZ-DisSW-1(config-if)# `exit`

    DMZ-DisSW-1(config)# `int range e2/0-1`

    DMZ-DisSW-1(config-if-range)# `channel-group 22 mode active`

    DMZ-DisSW-1(config-if-range)# `exit`

    DMZ-DisSW-1(config)# `int port-channel 22`

    DMZ-DisSW-1(config-if)# `Description Link to DMZ-AccSW-2`

    DMZ-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-DisSW-1(config-if)# `switchport mode trunk`

    DMZ-DisSW-1(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-DisSW-1(config-if)# `switchport nonegotiate`

    DMZ-DisSW-1(config-if)# `exit`

- **Configuration of the Distribution Switch interfaces for the connection to the second Distribution Switch**

    DMZ-DisSW-1(config)# `int range e3/0-1`

    DMZ-DisSW-1(config-if-range)# `channel-group 23 mode active`

    DMZ-DisSW-1(config-if-range)# `exit`

    DMZ-DisSW-1(config)# `int port-channel 23`

    DMZ-DisSW-1(config-if)# `Description Link to DMZ-DisSW-2`

    DMZ-DisSW-1(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-DisSW-1(config-if)# `switchport mode trunk`

    DMZ-DisSW-1(config-if)# `switchport trunk allowed vlan 50,70,80,100`

    DMZ-DisSW-1(config-if)# `switchport nonegotiate`

    DMZ-DisSW-1(config-if)# `exit`

- **Configuration of the Distribution Switch interfaces for the connection to the LAN-Firewalls**

    DMZ-DisSW-1(config)# `int e0/0`
    
    DMZ-DisSW-1(config-if)# `switchport mode access`
    
    DMZ-DisSW-1(config-if)# `switchport access vlan 50`
    
    DMZ-DisSW-1(config-if)# `exit`
    
    DMZ-DisSW-1(config)# `int e0/2`
    
    DMZ-DisSW-1(config-if)# `switchport mode access`
    
    DMZ-DisSW-1(config-if)# `switchport access vlan 50`
    
    DMZ-DisSW-1(config-if)# `exit`

- **Switching off the unuse interface**

    DMZ-DisSW-1(config)# `int range e1/2-3, e2/2-3, e3/2-3`

    DMZ-DisSW-1(config-if-range)# `shut`

    DMZ-DisSW-1(config-if-range)# `exit`

## <ins>Inter-VLAN routing configuration</ins>

The Distribution Switch 1 will be prioritized for the routing of VLAN 50, 70 and 100.

- **VLAN interface configuration for Inter-VLAN routing and HSRP**

    DMZ-DisSW-1(config)# `int vlan 50`

    DMZ-DisSW-1(config-if)# `ip add 172.16.50.11 255.255.255.248`

    DMZ-DisSW-1(config-if)# `no shut`

    DMZ-DisSW-1(config-if)# `standby 50 ip 172.16.50.13`

    DMZ-DisSW-1(config-if)# `standby 50 priority 150`

    DMZ-DisSW-1(config-if)# `standby 50 preempt`

    DMZ-DisSW-1(config-if)# `exit`

    DMZ-DisSW-1(config)# `int vlan 70`

    DMZ-DisSW-1(config-if)# `ip add 172.16.70.252 255.255.255.0`

    DMZ-DisSW-1(config-if)# `no shut`

    DMZ-DisSW-1(config-if)# `standby 70 ip 172.16.70.254`

    DMZ-DisSW-1(config-if)# `standby 70 priority 150`

    DMZ-DisSW-1(config-if)# `standby 70 preempt`

    DMZ-DisSW-1(config-if)# `exit`

    DMZ-DisSW-1(config)# `int vlan 80`

    DMZ-DisSW-1(config-if)# `ip add 172.16.80.252 255.255.255.0`

    DMZ-DisSW-1(config-if)# `no shut`

    DMZ-DisSW-1(config-if)# `standby 80 ip 172.16.80.254`

    DMZ-DisSW-1(config-if)# `exit`

    DMZ-DisSW-1(config)# `int vlan 100`

    DMZ-DisSW-1(config-if)# `ip add 172.16.100.252 255.255.255.128`

    DMZ-DisSW-1(config-if)# `no shut`

    DMZ-DisSW-1(config-if)# `standby 100 ip 172.16.100.254`

    DMZ-DisSW-1(config-if)# `standby 100 priority 150`

    DMZ-DisSW-1(config-if)# `standby 100 preempt`

    DMZ-DisSW-1(config-if)# `exit`

## <ins>Define ACL for inter-VLAN Routing</ins>

...

## <ins>Spanning-Tree Configuration</ins>

- **Define the Spanning-Tree priority for choosing the root switch**

    The same choice for the switch priority have been taken for spanning-tree and inter-vlan routing. 

    DMZ-DisSW-1(config)# `spanning-tree vlan 70 root primary`

    DMZ-DisSW-1(config)# `spanning-tree vlan 80 root secondary`

    DMZ-DisSW-1(config)# `spanning-tree vlan 100 root primary`