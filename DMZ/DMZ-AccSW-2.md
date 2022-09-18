# DMZ-AccSW-2 configuration

## <ins>Initial Configuration</ins>

The naming convention of the hardware is [zone location]-\[architecture location][Hardware Type]-\[number].

- **Defining the hardware hostname**

    ios> `en`

    ios# `conf t`

    ios(config)# `hostname DMZ-AccSW-2`

## <ins>Login Credentials</ins>

Even if it is not a production project, I like to get into the good habit of not exposing passwords and using the environment variables made available by Github.

- **Control access for enable (privileged) mode use of scrypt as the hashing algorithm**

    DMZ-AccSW-2(config)# `enable algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Defining username/secret(password) for authentification**

    DMZ-AccSW-2(config)# `username admin algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Configuring the command line to use locally configured username/secret pairs**

    DMZ-AccSW-2(config)# `line con 0`

    DMZ-AccSW-2(config-line)# `login local`

    DMZ-AccSW-2(config-line)# `exit`

- **Configuring the terminal access to use locally configured username/secret pairs**

    DMZ-AccSW-2(config)# `line vty 0 4`

    DMZ-AccSW-2(config-line)# `login local`

    DMZ-AccSW-2(config-line)# `exit`

## <ins>VLANs Configuration</ins>

This switch is connected to the different servers. 
VLAN 70 for the forwarding Proxy and DNS public resolution. 
VLAN 80 for the reverse Proxy/Load Balancer and Web servers.
And finally, VLAN 100 for management purpose.

- **Configuring the VLANs name**

    DMZ-AccSW-2(config)# `vlan 70`

    DMZ-AccSW-2(config-vlan)# `name Internet`

    DMZ-AccSW-2(config-vlan)# `vlan 80`

    DMZ-AccSW-2(config-vlan)# `name Public`

    DMZ-AccSW-2(config-vlan)# `vlan 100`

    DMZ-AccSW-2(config-vlan)# `name Management`

    DMZ-AccSW-2(config-vlan)# `exit`

## <ins>Interface Configuration</ins>

- **Configuring the switch interfaces for the endpoints**

    DMZ-AccSW-2(config)# `int range e2/0-1`

    DMZ-AccSW-2(config-if-range)# `switchport mode access`

    DMZ-AccSW-2(config-if-range)# `switchport access vlan 80`

    DMZ-AccSW-2(config-if-range)# `spanning-tree portfast`

    DMZ-AccSW-2(config-if-range)# `spanning-tree bpduguard enable`

    DMZ-AccSW-2(config-if-range)# `exit`

- **Configuring the switch interfaces of the Access Switch for the connection to the Distribution switch (use of etherchannel to increase the bandwith)**

    DMZ-AccSW-(config)# `int range e0/0-1`

    DMZ-AccSW-(config-if-range)# `channel-group 22 mode active`

    DMZ-AccSW-(config-if-range)# `exit`

    DMZ-AccSW-(config)# `int port-channel 22`

    DMZ-AccSW-(config-if)# `Description Link to DMZ-DisSW-1`

    DMZ-AccSW-(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-AccSW-(config-if)# `switchport mode trunk`

    DMZ-AccSW-(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-AccSW-(config-if)# `switchport nonegotiate`

    DMZ-AccSW-(config-if)# `exit`

    DMZ-AccSW-(config)# `int range e1/0-1`

    DMZ-AccSW-(config-if-range)# `channel-group 25 mode active`

    DMZ-AccSW-(config-if-range)# `exit`

    DMZ-AccSW-(config)# `int port-channel 25`

    DMZ-AccSW-(config-if)# `Description Link to DMZ-DisSW-2`

    DMZ-AccSW-(config-if)# `switchport trunk encapsulation dot1q`

    DMZ-AccSW-(config-if)# `switchport mode trunk`

    DMZ-AccSW-(config-if)# `switchport trunk allowed vlan 70,80,100`

    DMZ-AccSW-(config-if)# `switchport nonegotiate`

    DMZ-AccSW-(config-if)# `exit`

- **Switching off the unuse interface**

    DMZ-AccSW-(config)# `int range e0/2-3, e1/2-3, e2/2-3, e3/0-3`

    DMZ-AccSW-(config-if-range)# `shut`

    DMZ-AccSW-(config-if-range)# `exit`

- **Configuring the interface of vlan 100 for management purpose**

    DMZ-AccSW-(config)# `int vlan 100`

    DMZ-AccSW-(config-if)# `ip add 172.16.100.130 255.255.255.128`

    DMZ-AccSW-(config-if)# `no shut`

    DMZ-AccSW-(config-if)# `exit`

    DMZ-AccSW-(config)# `ip route 0.0.0.0 0.0.0.0 172.16.100.254`

    DMZ-AccSW-(config)# `end`