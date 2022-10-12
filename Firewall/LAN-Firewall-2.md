# LAN-Firewall-2 Configuration

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

## <ins>High Availability Configuration</ins>

- **Setup failover interface on the secondary firewall**

    LAN-Firewall(config)# `failover lan unit secondary`

    LAN-Firewall(config)# `int g0/2`

    LAN-Firewall(config)# `no shut`

- **Assign the failover Ip-Address on the secondary firewall (FAILOVER will be the name assigned to the interface).**

    LAN-Firewall(config)# `failover lan interface FAILOVER g0/2`

    LAN-Firewall(config)# `failover interface ip FAILOVER 172.16.55.1 255.255.255.252 standby 172.16.55.2`

    LAN-Firewall(config)# `failover key ${{secrets.CREDENTIAL_SECRET}}`

    LAN-Firewall(config)# `failover link FAILOVER`

    LAN-Firewall(config)# `failover`

- **Each time a configuration will be saved on the primary (LAN-Firewall-1), the secundary/mate (LAN-Firewall) will received it.**

    LAN-Firewall/stby(config)#






