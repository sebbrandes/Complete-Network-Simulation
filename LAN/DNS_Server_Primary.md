# DNS Configuration
---

The DNS service is already installed in the Docker image (debian-server).
The server will have as an IP address : 172.16.10.11/24 (gateway : 172.16.10.254).
It's a DNS for intern resolution query (private). For externe resolution query (public), it will forward to the DNS server in the DMZ.

## Configuration of DNS server's options

Code BASH :

    sudo vim /etc/bind/named.conf.options

Adding the following lines in this file :

    options {
            directory "/var/cache/bind";

            // forward address of the DNS in DMZ
            
            forwarders { 172.16.70.1; };

            // Port exchange between DNS servers

            query-source address * port * ;

            // BIND sends a NXDOMAIN response in case the domain referenced
            // in the query does not exist.
            // If auth-nxdomain is set to yes, the answer will me marked as "authoritative"
            // even if it is not the authoritative server for the corresponding zone

            auth-nxdomain no;

            // Listen on local interfaces only

    	    listen-on {127.0.0.1; 172.16.10.11; };

            // Transfer zone information to secondary DNS

	        allow-transfer { 172.16.10.12; };

            // Accept requests for internal network only

	        allow-query {172.16.0.0/16; localnets; };

            // Allow recursive queries for local hosts

	        allow-recursion { 172.16.0.0/16; localnets; };

            // Do not make BIND version public

	        version none;        


    };


## Creation of a DNS zone file

CODE BASH :

    sudo vim /etc/bind/db.myenterprise.com

In this file, adding the following lines :

    $TTL 3600

    @	    IN	SOA	ns1.myenterprise.com.	root.myenterprise.com.	(
                    2022091001	; Serial
                    3600		; Refresh (1 h)
                    600		; Retry (10 min)
                    86400		; Expire (1 day)
                    3600 )		; Minimum (1 h)
    ;

    @	    IN	NS	ns1	;
    @	    IN	NS	ns2	;
    ns1	    IN	A	172.16.10.11	;	IP name server principal
    ns2	    IN	A	172.16.10.12	;	IP name server secondary
    dhcp	IN	A	172.16.10.1	    ;	IP DHCP server

## Configure address to name mappings

CODE BASH :

    sudo vim /etc/bind/db.myenterprise.inv

In this file, adding the following lines :

    $TTL 3600

    @	    IN	SOA	ns1.myenterprise.com.	root.myenterprise.com.	(
                    2022091001	; Serial
                    3600		; Refresh (1 h)
                    600		; Retry (10 min)
                    86400		; Expire (1 day)
                    3600 )		; Minimum
    ;

        @	IN	NS	ns1.myenterprise.com.	;
        @	IN	NS	ns2.myenterprise.com.	;

    11.10	IN	PTR	ns1.myenterprise.com.	;
    12.10	IN	PTR	ns2.myenterprise.com.	;
    1.10	IN	PTR	dhcp.myenterprise.com.	;

## Updating the BIND configuration file

Insert both zones file names into the BIND configuration file

CODE BASH :

    sudo vim /etc/bind/named.conf.local

In this file, insert both zone file names:

    zone "myenterprise.com" {
        type master;
        file "/etc/bind/db.myenterprise.com";
        notify no;
    };

    zone "16.172.in-addr.arpa" {
        type master;
        file "/etc/bind/db.myenterprise.com.inv";
        notify no;
    };

## Checking BIND's zone files and configuration

To check the configuration files, run the following command 

    sudo named-checkconf -z

To check the DNS zones files, the command `named-checkzone` will be used :

    sudo named-checkzone myenterprise.com /etc/bind/db.myenterprise.com

To check the reverse zone file :

    sudo named-checkzone 16.172.in-addr.arpa /etc/bind/db.myenterprise.com.inv

## Start BIND nameserver

    sudo /etc/init.d/named start
