## Generate a [dynamic self-signed certificate](https://wiki.squid-cache.org/Features/DynamicSslCert) 

    openssl req -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -extensions v3_ca -keyout myCA.pem -out myCA.pem



## Squid Configuration

Code BASH :

    mv /etc/squid/squid.conf /etc/squid/squid-default.conf

    nano /etc/squid/squid.conf

In the file, adding the following line :

    #List our internal IP networks from where browsing should be allowed
    acl developer src 172.16.30.0/24 # VLAN 30
    acl office src 172.16.40.0/24 # VLAN 40

    acl SSL_ports port 443
    acl Safe_ports port 80          # http
    acl Safe_ports port 21          # ftp
    acl Safe_ports port 443         # https
    acl Safe_ports port 1025-65535  # unregistered ports
    acl Safe_ports port 777         # multiling http
    acl CONNECT method CONNECT
    acl PURGE method PURGE

    # Deny requests to certain unsafe ports
    http_access deny !Safe_ports

    # Deny CONNECT to other than secure SSL ports
    http_access deny connect !SSL_ports

    # INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS

    # Adapt localnet in the ACL section to list your (internal) IP networks
    # from where browsing should be allowed 
    http_access allow developer
    http_access allow office

    # Allow developer (Dev - VLAN 30) to purge an object from the Squid cache
    http_access allow PURGE developer 
    http_access deny PURGE

    # And finally deny all other access to this proxy
    http_access deny all

    http_port 172.16.70.1:3128
    https_port 172.16.70.1:3129 intercept ssl-bump cert=/etc/squid/cert/myCA.pem generate-host-certificates=on dynamic_cert_mem_cache_size=4MB
    acl step1 at_step SslBump1
    ssl_bump peek step1
    ssl_bump bump all
    ssl_bump splice all

    # Add a disk cache directory 

    cache_dir ufs /var/spool/squid 100 16 256

    # Leave coredumps in the first cache dir

    coredump_dir /var/spool/squid