# DNS Configuration
---

The DNS service is already installed in the Docker image (debian-server).
The server will have as an IP address : 172.16.10.12/24 (gateway : 172.16.10.254).

## Updating the BIND configuration file

Insert both zones file names into the BIND configuration file

CODE BASH :

    sudo vim /etc/bind/named.conf.local

In this file, add the following lines with the type slave directive indicating the secondary server and the masters directive with the address of the primary server:

    zone "myenterprise.com" {
        type slave;
        file "/etc/bind/db.myenterprise.com";
        masters { 172.16.10.11; };
    };

    zone "16.172.in-addr.arpa" {
        type slave;
        file "/etc/bind/db.myenterprise.com.inv";
        masters { 172.16.10.11; };
    };    

## Configuration of DNS server's options

Code BASH :

    sudo vim /etc/bind/named.conf.options

Allow notification from the primary DNS server by adding the following line with the address of the primary DNS server:

    allow-notify {172.16.10.11; }

## Start BIND nameserver

    sudo /etc/init.d/named start
