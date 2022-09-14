# LAN-AccSW-4 Configuration

## <ins>Initial Configuration</ins>

The naming convention of the hardware is [zone location]-\[architecture location][Hardware Type]-\[number].

- **Defining the hardware hostname**

    ios> `en`

    ios# `conf t`

    ios(config)# `hostname LAN-AccSW-4`


## <ins>Login Credentials</ins>

Even if it is not a production project, I like to get into the good habit of not exposing passwords and using the environment variables made available by Github.

- **Control access for enable (privileged) mode use of scrypt as the hashing algorithm**

    LAN-AccSW-4(config)# `enable algorithm-type secret scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Defining username/secret(password) for authentification**

    LAN-AccSW-4(config)# `username admin algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Configuring the command line to use locally configured username/secret pairs**

    LAN-AccSW-4(config)# `line con 0`

    LAN-AccSW-4(config-line)# `login local`

    LAN-AccSW-4(config-line)# `exit`

- **Configuring the terminal access to use locally configured username/secret pairs**

    LAN-AccSW-4(config)# `line vty 0 4`

    LAN-AccSW-4(config-line)# `login local`

    LAN-AccSW-4(config-line)# `exit`

## <ins>VLANs Configuration</ins>

This switch is connected to the endpoint in VLAN 40. 
The VLAN 100 is choosen for management purpose (permit access to the configuration of the hardware components accross the LAN network).

-  **Configuring the VLANs name**

    LAN-AccSW-4(config)# `vlan 40`

    LAN-AccSW-4(config-vlan)# `name Office`

    LAN-AccSW-4(config-vlan)# `vlan 100`

    LAN-AccSW-4(config-vlan)# `name Management`

    LAN-AccSW-4(config-vlan)# `exit`

## <ins>Interface Configuration</ins>

- **Configuring the switch interfaces for the endpoints**

    LAN-AccSW-4(config)# `int range e0/0`

    LAN-AccSW-4(config-if)# `switchport mode access`

    LAN-AccSW-4(config-if)# `switchport access vlan 40`

    LAN-AccSW-4(config-if)# `spanning-tree portfast`

    LAN-AccSW-4(config-if)# `spanning-tree bpduguard enable`

    LAN-AccSW-4(config-if)# `exit`


- **Configuring the switch interfaces of the Access Switch for the connection to the Distribution switch (use of etherchannel to increase the bandwith)**

    LAN-AccSW-4(config)# `int range e2/0-1`

    LAN-AccSW-4(config-if-range)# `channel-group 7 mode active`

    LAN-AccSW-4(config-if-range)# `exit`

    LAN-AccSW-4(config)# `int port-channel 7`

    LAN-AccSW-4(config-if)# `Description Link to LAN-DisSW-1`

    LAN-AccSW-4(config-if)# `switchport trunk encapsulation dot1q`

    LAN-AccSW-4(config-if)# `switchport mode trunk`

    LAN-AccSW-4(config-if)# `switchport trunk allowed vlan 40,100`

    LAN-AccSW-4(config-if)# `switchport nonegotiate`

    LAN-AccSW-4(config-if)# `exit`

    LAN-AccSW-4(config)# `int range e3/0-1`

    LAN-AccSW-4(config-if-range)# `channel-group 8 mode active`

    LAN-AccSW-4(config-if-range)# `exit`

    LAN-AccSW-4(config)# `int port-channel 8`

    LAN-AccSW-4(config-if)# `Description Link to LAN-DisSW-2`

    LAN-AccSW-4(config-if)# `switchport trunk encapsulation dot1q`

    LAN-AccSW-4(config-if)# `switchport mode trunk`

    LAN-AccSW-4(config-if)# `switchport trunk allowed vlan 40,100`

    LAN-AccSW-4(config-if)# `switchport nonegotiate`

    LAN-AccSW-4(config-if)# `exit`

- **Switching off the unuse interface**

    LAN-AccSW-4(config)# `int range e0/1-3, e1/0-3, e2/2-3, e3/2-3`

    LAN-AccSW-4(config-if-range)# `shut`

    LAN-AccSW-4(config-if-range)# `exit`

- **Configuring the interface of vlan 100 for management purpose**

    LAN-AccSW-4(config)# `int vlan 100`

    LAN-AccSW-4(config-if)# `ip add 172.16.100.4 255.255.255.0`

    LAN-AccSW-4(config-if)# `no shut`

    LAN-AccSW-4(config-if)# `exit`

    LAN-AccSW-4(config)# `ip route 0.0.0.0 0.0.0.0 172.16.100.254`

    LAN-AccSW-4(config)# `end`








