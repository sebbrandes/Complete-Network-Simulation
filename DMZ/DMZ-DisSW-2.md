# DMZ-DisSW-2 configuration

## <ins>Initial Configuration</ins>

The naming convention of the hardware is [zone location]-\[architecture location][Hardware Type]-\[number].

- **Define the hardware hostname**

    ios> `en`

    ios# `conf t`

    ios(config)# `hostname DMZ-DisSW-2`

## <ins>Login Credentials</ins>

Even if it is not a production project, I like to get into the good habit of not exposing passwords and using the environment variables made available by Github.

- **Control access for enable (privileged) mode use of scrypt as the hashing algorithm**

    DMZ-DisSW-2(config)# `enable algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Define username/secret(password) for authentification**

    DMZ-DisSW-2(config)# `username admin algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Command line configuration to use locally configured username/secret pairs**

    DMZ-DisSW-2(config)# `line con 0`

    DMZ-DisSW-2(config-line)# `login local`

    DMZ-DisSW-2(config-line)# `exit`

- **Terminal access configuration to use locally configured username/secret pairs**

    DMZ-DisSW-2(config)# `line vty 0 4`

    DMZ-DisSW-2(config-line)# `login local`

    DMZ-DisSW-2(config-line)# `exit`

## <ins>VLANs Configuration</ins>

This switch is connected to the different servers. 
VLAN 70 for the forwarding Proxy and DNS public resolution. 
VLAN 80 for the reverse Proxy/Load Balancer and Web servers.
And finally, VLAN 100 for management purpose.

- **Define the VLANs name**

    DMZ-DisSW-2(config)# `vlan 50`

    DMZ-DisSW-2(config-vlan)# `name Network`

    DMZ-DisSW-2(config)# `vlan 70`

    DMZ-DisSW-2(config-vlan)# `name Internet`

    DMZ-DisSW-2(config-vlan)# `vlan 80`

    DMZ-DisSW-2(config-vlan)# `name Public`

    DMZ-DisSW-2(config-vlan)# `vlan 100`

    DMZ-DisSW-2(config-vlan)# `name Management`

    DMZ-DisSW-2(config-vlan)# `exit`

## <ins> Interface Configuration</ins>

- **Configuration of the Distribution Switch interfaces for the connection to the Acces switch (use of etherchannel to increase the bandwith)**

    DMZ-DisSW-2(config)# `int range e1/0-1`

    DMZ-DisSW-2(config-if-range)# `channel-group 24 mode active`

    DMZ-DisSW-2(config-if-range)# `exit`

    DMZ-DisSW-2(config)# `int port-channel 24`

    DMZ-DisSW-2(config-if)# `Description Link to DMZ-AccSW-1`

    DMZ-DisSW-2(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-DisSW-2(config-if)# `switchport mode trunk`

    DMZ-DisSW-2(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-DisSW-2(config-if)# `switchport nonegotiate`

    DMZ-DisSW-2(config-if)# `exit`

    DMZ-DisSW-2(config)# `int range e2/0-1`

    DMZ-DisSW-2(config-if-range)# `channel-group 25 mode active`

    DMZ-DisSW-2(config-if-range)# `exit`

    DMZ-DisSW-2(config)# `int port-channel 25`

    DMZ-DisSW-2(config-if)# `Description Link to DMZ-AccSW-2`

    DMZ-DisSW-2(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-DisSW-2(config-if)# `switchport mode trunk`

    DMZ-DisSW-2(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-DisSW-2(config-if)# `switchport nonegotiate`

    DMZ-DisSW-2(config-if)# `exit`

- **Configuration of the Distribution Switch interfaces for the connection to the second Distribution Switch**

    DMZ-DisSW-2(config)# `int range e3/0-1`

    DMZ-DisSW-2(config-if-range)# `channel-group 23 mode active`

    DMZ-DisSW-2(config-if-range)# `exit`

    DMZ-DisSW-2(config)# `int port-channel 23`

    DMZ-DisSW-2(config-if)# `Description Link to DMZ-DisSW-2`

    DMZ-DisSW-2(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-DisSW-2(config-if)# `switchport mode trunk`

    DMZ-DisSW-2(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-DisSW-2(config-if)# `switchport nonegotiate`

    DMZ-DisSW-2(config-if)# `exit`

- **Configuration of the Distribution Switch interfaces for the connection to the LAN-Firewalls**

    DMZ-DisSW-2(config)# `int e0/0`

    DMZ-DisSW-2(config-if)# `switchport mode access`

    DMZ-DisSW-2(config-if)# `switchport access vlan 50`

    DMZ-DisSW-2(config-if)# `exit`

    DMZ-DisSW-2(config)# `int e0/2`

    DMZ-DisSW-2(config-if)# `switchport mode access`

    DMZ-DisSW-2(config-if)# `switchport access vlan 50`

    DMZ-DisSW-2(config-if)# `exit`

- **Switching off the unuse interface**

    DMZ-DisSW-2(config)# `int range e1/2-3, e2/2-3, e3/2-3`

    DMZ-DisSW-2(config-if-range)# `shut`

    DMZ-DisSW-2(config-if-range)# `exit`

## <ins>Inter-VLAN routing configuration</ins>

The Distribution Switch 2 will be prioritized for the routing of VLAN 80.

- **VLAN interfaces configuration for Inter-VLAN routing and HSRP**

    DMZ-DisSW-2(config)# `int vlan 50`

    DMZ-DisSW-2(config-if)# `ip add 172.16.50.12 255.255.255.248`

    DMZ-DisSW-2(config-if)# `no shut`

    DMZ-DisSW-2(config-if)# `standby 50 ip 172.16.50.13`

    DMZ-DisSW-2(config-if)# `exit`

    DMZ-DisSW-2(config)# `int vlan 80`

    DMZ-DisSW-2(config-if)# `ip add 172.16.80.253 255.255.255.0`

    DMZ-DisSW-2(config-if)# `no shut`

    DMZ-DisSW-2(config-if)# `standby 80 ip 172.16.80.254`

    DMZ-DisSW-2(config-if)# `standby 80 priority 150`

    DMZ-DisSW-2(config-if)# `standby 80 preempt`

    DMZ-DisSW-2(config-if)# `exit`

    DMZ-DisSW-2(config)# `int vlan 70`

    DMZ-DisSW-2(config-if)# `ip add 172.16.70.253 255.255.255.0`

    DMZ-DisSW-2(config-if)# `no shut`

    DMZ-DisSW-2(config-if)# `standby 70 ip 172.16.70.254`

    DMZ-DisSW-2(config-if)# `exit`

    DMZ-DisSW-2(config)# `int vlan 100`

    DMZ-DisSW-2(config-if)# `ip add 172.16.100.253 255.255.255.128`

    DMZ-DisSW-2(config-if)# `no shut`

    DMZ-DisSW-2(config-if)# `standby 100 ip 172.16.100.254`

    DMZ-DisSW-2(config-if)# `exit`

## <ins>Define the ACL for inter-VLAN Routing</ins>

...

## <ins>Spanning-Tree Configuration</ins>

- **Define the Spanning-Tree priority for choosing the root switch**

    The same choice for the switch priority have been taken for spanning-tree and inter-vlan routing. 

    DMZ-DisSW-2(config)# `spanning-tree vlan 70 root secondary`

    DMZ-DisSW-2(config)# `spanning-tree vlan 80 root primary`

    DMZ-DisSW-2(config)# `spanning-tree vlan 100 root secondary`