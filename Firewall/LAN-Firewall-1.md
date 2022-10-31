# LAN-Firewall-1 Configuration

## <ins>Initial Configuration</ins>

The naming convention of the hardware is [zone location]-\[Hardware Type]

- **Define the hardware hostname**

    ciscoasa# `conf t`
   
    ciscoasa(config)# `hostname LAN-Firewall`

## <ins>Login Credentials</ins>

Even if it is not a production project, I like to get into the good habit of not exposing passwords and using the environment variables made available by Github.

- **Control access for enable (privileged) mode use of scrypt as the hashing algorithm**

    LAN-Firewall(config)# `enable algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Set username/secret(password) for authentification**

    LAN-Firewall(config)# `username admin algorithm-type scrypt secret ${{secrets.CREDENTIAL_SECRET}}`

- **Command line configuration to use locally configured username/secret pairs**

    LAN-Firewall(config)# `line con 0`

    LAN-Firewall(config-line)# `login local`

    LAN-Firewall(config-line)# `exit`

- **Terminal access configuration to use locally configured username/secret pairs**

    LAN-Firewall(config)# `line vty 0 4`

    LAN-Firewall(config-line)# `login local`

    LAN-Firewall(config-line)# `exit`

## <ins>Interface Configuration</ins>

- **Interfaces in the LAN area :**

    LAN-Firewall(config)# `int g0/0`

    LAN-Firewall(config-if)# `no shut`

    LAN-Firewall(config-if)# `exit`

    LAN-Firewall(config)# `int g0/1`

    LAN-Firewall(config-if)# `no shut`

    LAN-Firewall(config-if)# `exit`

    LAN-Firewall(config)# `interface redundant 1`

    LAN-Firewall(config-if)# `member-interface g0/0`

    LAN-Firewall(config-if)# `member-interface g0/1`

    LAN-Firewall(config-if)# `nameif LAN`

    LAN-Firewall(config-if)# `security-level 80`

    LAN-Firewall(config-if)# `ip add 172.16.50.5 
    255.255.255.248 standby 172.16.50.6`

    Enable OSPF MD5 authentication

    LAN-Firewall(config-if)# `ospf message-digest-key 1 md5 secret`

    LAN-Firewall(config-if)# `exit`


- **Interfaces in the DMZ area :**

    LAN-Firewall(config)# `int g0/3`

    LAN-Firewall(config-if)# `no shut`

    LAN-Firewall(config-if)# `exit`

    LAN-Firewall(config)# `int g0/4`

    LAN-Firewall(config-if)# `no shut`

    LAN-Firewall(config-if)# `exit`
    
    LAN-Firewall(config)# `interface redundant 2`

    LAN-Firewall(config-if)# `member-interface g0/3`

    LAN-Firewall(config-if)# `member-interface g0/4`

    LAN-Firewall(config-if)# `nameif DMZ`

    LAN-Firewall(config-if)# `security-level 50`

    LAN-Firewall(config-if)# `ip add 172.16.50.9 
    255.255.255.248 standby 172.16.50.10`

    Enable OSPF MD5 authentication

    LAN-Firewall(config-if)# `ospf message-digest-key 1 md5 secret`

    LAN-Firewall(config-if)# `exit`

- **Interfaces in the ADMIN area :**

    LAN-Firewall(config)# `int g0/5`

    LAN-Firewall(config-if)# `no shut`

    LAN-Firewall(config-if)# `exit`

    LAN-Firewall(config)# `int g0/6`

    LAN-Firewall(config-if)# `no shut`

    LAN-Firewall(config-if)# `exit`

    LAN-Firewall(config)# `interface redundant 3`

    LAN-Firewall(config-if)# `member-interface g0/5`

    LAN-Firewall(config-if)# `member-interface g0/6`

    LAN-Firewall(config-if)# `nameif ADMIN`

    LAN-Firewall(config-if)# `security-level 100`

    LAN-Firewall(config-if)# `ip add 172.16.60.5 255.255.255.248 standby 172.16.60.6`

    Enable OSPF MD5 authentication

    LAN-Firewall(config-if)# `ospf message-digest-key 1 md5 secret`

    LAN-Firewall(config-if)# `exit`

    
## <ins>High Availability Configuration</ins>

- **Setup failover interface on the primary firewall**

    LAN-Firewall(config)# `failover lan unit primary`

    LAN-Firewall(config)# `int g0/2`

    LAN-Firewall(config)# `no shut`

- **Assign the failover Ip-Address on the primary firewall (FAILOVER will be the name assigned to the interface).** 

    LAN-Firewall(config)# `failover lan interface FAILOVER g0/2`

    LAN-Firewall(config)# `failover interface ip FAILOVER 172.16.55.1 255.255.255.252 standby 172.16.55.2`

    LAN-Firewall(config)# `failover key ${{secrets.CREDENTIAL_SECRET}}`

    LAN-Firewall(config)# `failover link FAILOVER`

    LAN-Firewall(config)# `failover`
     
- **Prompt Configuration**

    Since the two firewalls share the same configuration, it is no longer possible to distinguish between them using the hostname. The prompt will be configured to show if the firewall is active or on standby.

    LAN-Firewall(config)# `prompt hostname state`

    LAN-Firewall/act(config)# `exit`

    LAN-Firewall/act# `copy run start`



## <ins>Routing Configuration</ins>

- **Enable OSPF and specify networks**

    LAN-Firewall/act# `conf t`

    LAN-Firewall/act(config)# `router opsf 1`

    LAN-Firewall/act(config-router)# `network 172.16.50.0 0.0.0.255 area 0`

    LAN-Firewall/act(config-router)# `network 172.16.60.0 0.0.0.255 area 0`

    The area authentication message-digest command in this configuration enables authentications for all of the router interfaces in a particular area

    LAN-Firewall/act(config-router)# `area 0 authentication message-digest`

    LAN-Firewall/act(config-router)# `exit`

## <ins>Firewall's Rule</ins>

### ICMP

The implementation of a stateful rule to allow ICMP on the Cisco ASA firewall is realized in two steps :

1. Creation of a specific ACL on an incoming interface

2. Enable "inspect icmp" to dynamically create an ACL allowing the reply without the need to implement an ACL on the interface receiving the reply (This configuration is recommended because dynamic ACLs are generated per session "as needed" basis, and will be removed after timeout values expires)

Network's ICMP : 

- **ACL for allowing ICMP between the network hardware**

    LAN-Firewall/act(config)# `access-list LAN_IN remark icmp between network hardware`

    LAN-Firewall/act(config)# `access-list LAN_IN extended permit icmp 172.16.50.0 255.255.255.248 172.16.50.8 255.255.255.248`


LAN's ICMP

The rules are implemented for allowing the test of the connectivity from the LAN. The other way will not be configured to prevent network discovery by an attacker if the DMZ was compromised.
Only VLAN 30 (Developer) and VLAN 40 need an access to the DMZ (Services and Backup are restricted to the LAN).

- **ACL for allowing ICMP from the VLAN 30 (Developer) to network hardware in the DMZ**

    LAN-Firewall/act(config)# `access-list LAN_IN remark icmp between Dev and the hardware network`

    LAN-Firewall/act(config)# `access-list LAN_IN extended permit icmp 172.16.30.0 255.255.255.0 172.16.50.8 255.255.255.248`
    
- **ACL for allowing ICMP from the VLAN 30 (Developer) to Proxy (VLAN 70) in the DMZ**

    LAN-Firewall/act(config)# `access-list LAN_IN remark icmp between Dev and Proxy`
    
    LAN-Firewall/act(config)# `access-list LAN_IN extended permit icmp 172.16.30.0 255.255.255.0 172.16.70.0 255.255.255.0`

- **ACL for allowing ICMP from the VLAN 40 (Office) to network hardware in the DMZ**

    LAN-Firewall/act(config)# `access-list LAN_IN remark icmp between Office and the hardware network`

    LAN-Firewall/act(config)# `access-list LAN_IN extended permit icmp 172.16.40.0 255.255.255.0 172.16.50.8 255.255.255.248`

- **ACL for allowing ICMP from the VLAN 40 (Office) to Proxy (VLAN 70) in the DMZ**

    LAN-Firewall/act(config)# `access-list LAN_IN remark icmp between Office and the Proxy`

    LAN-Firewall/act(config)# `access-list LAN_IN extended permit icmp 172.16.40.0 255.255.255.0 172.16.70.0 255.255.255.0`

 - **Apply the ACL on the interface on the LAN side**

    LAN-Firewall/act(config)# `access-group LAN_IN in interface LAN`


ADMIN's ICMP :

- **ACL for allowing ICMP from the ADMIN zone to all the enterprise network**

    LAN-Firewall/act(config)# `access-list ADMIN_IN remark icmp between ADMIN zone and the enterprise network`

    LAN-Firewall/act(config)# `access-list ADMIN_IN extended permit icmp 172.16.60.0 255.255.255.0 172.16.0.0 255.255.0.0` 

    LAN-Firewall/act(config)# `access-group ADMIN_IN in interface ADMIN`

- **Enabling inspect icmp on the ASA Firewall to allow the firewall to dynamically create ACLs**

    LAN-Firewall/act(config)# `policy-map global_policy`

    LAN-Firewall/act(config-pmap)# `class inspection_default`

    LAN-Firewall/act(config-pmap-c)# `inspect icmp`

## Final

- **Save the configuration on the primary and synchronise it with the secundary/mate.**

    LAN-Firewall/act# `copy run start`

