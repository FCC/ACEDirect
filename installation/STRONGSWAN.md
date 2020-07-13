# strongSwan

strongSwan is an open-source VPN software that is widely popular within the IPSec industry. strongSwan can be installed on both CentOS and Amazon Linux servers; the below instructions were implemented on an Amazon Linux EC2 instance.

## Installation Process

First, use yum to install strongSwan:

    yum install -y strongswan

Then, modify the /etc/strongswan/ipsec.conf table to reflect the following:

    config setup
            #charondebug="ike 2, net 3, knl 2, cfg 2"    #useful debugs


    conn %default
            ikelifetime=480m
            keylife=60m
            rekeymargin=3m
            keyingtries=1
            keyexchange=ikev1
            authby=secret

    conn iConectiv
            auto=start
            ike=aes128-sha1-modp1024        #P1: modp1536 = DH group 2
            esp=aes128-sha1-modp1024        #P2

            left=<StongSWAN local address>  #MITRE outside address
            leftsubnet=<dummy address>/32   #network behind MITRE
            leftid=<strongSwan public IP>   #IKEID sent by MITRE
            leftfirewall=yes

            right=<encrypted domain>         #iConectiv outside address
            rightsubnet=<VPN Tunnel IP>/32   #network behind iConectiv
            rightid=<encrypted domain>       #IKEID sent by iConectiv

Modify the /etc/strongswan/ipsec.secrets file to reflect the following:

    <strongSwan public IP> <rightsubnet> : PSK "<Key obtained from Neustar>"

Create the /etc/iptables.conf file and add the following:

        *nat
        :PREROUTING ACCEPT [5:436]
        :INPUT ACCEPT [1:92]
        :OUTPUT ACCEPT [34:9996]
        :POSTROUTING ACCEPT [34:9996]
        -A POSTROUTING -s <VPC CIDR block>/24 -d <rightsubnet> -o eth0 -j SNAT --to-source <dummy address>
        COMMIT


        *filter
        :INPUT ACCEPT [1063:95316]
        :FORWARD ACCEPT [12:1032]
        :OUTPUT ACCEPT [1018:375057]
        COMMIT

Append the following lines to /etc/rc.d/rc.local:

        /sbin/modprobe dummy
        /sbin/ifconfig dummy0 <dummy address> netmask 255.255.255.0
        /sbin/iptables-restore < /etc/iptables.conf

Move into the /etc/rc3.d directory and create the following symbolic links so that strongSwan gracefully starts/stops when the EC2 instance is rebooted:

        cd /etc/rc3.d
        ln -s  ../init.d/strongswan S48strongswan
        ln -s  ../init.d/strongswan K52strongswan

Modify the below variable within the /etc/sysctl.conf file to the following:

        net.ipv4.ip_forward = 1

iConectiv will automatically send a DELETE signal to strongSwan every 8 hours, causing the tunnel to close. In order to mitigate this, create the following script in /root/scripts:

        #!/bin/bash

        DEBUG=true
        LOG=/var/log/itrsmon.log

        if ! dig @<rightsubnet> in naptr 2.2.2.2.1.1.1.5.5.5.1.itrs.us>/dev/null 2>&1
        then
        echo $(/bin/date) " - ITRS Lookup failed" >> $LOG
        /etc/init.d/strongswan restart
        else
        $DEBUG && echo $(/bin/date) " - ITRS Lookup success" >> $LOG
        fi

Add the following line to the crontab:

        $ crontab -e


        * * * * * /root/scripts/itrs_mon.sh

Finally, reboot the instance to verify that the strongSwan service is running. Once it is, you can move onto the "AWS Specific Config" section below if you are in AWS. Once those steps are complete, DNS queries to the iTRS database from Asterisk servers should be successful. An example of a successful iTRS query is below:

        $ dig @<rightsubnet> in naptr 2.2.2.2.1.1.1.5.5.5.1.itrs.us
        ; <<>> DiG 9.9.4-RedHat-9.9.4-50.el7_3.1 <<>> @<rightsubnet> in naptr 2.2.2.2.1.1.1.5.5.5.1.itrs.us
        ; (1 server found)
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32364
        ;; flags: qr aa rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
        ;; WARNING: recursion requested but not available

        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 5120
        ;; QUESTION SECTION:
        ;2.2.2.2.1.1.1.5.5.5.1.itrs.us. IN NAPTR

        ;; ANSWER SECTION:
        2.2.2.2.1.1.1.5.5.5.1.itrs.us. 900 IN NAPTR 10 1 "u" "E2U+h323" "!^(.*)$!h323:\\1@192.168.1.1!" .

        ;; Query time: 15 msec
        ;; SERVER: <rightsubnet>#53(<rightsubnet>)
        ;; WHEN: Fri Aug 04 14:40:59 UTC 2017
        ;; MSG SIZE rcvd: 115

## AWS Specific Config

### VPC Route Table

A route from the strongSwan must be created in the VPC route table so that traffic to the iTRS database is routed to the VPN instance (which in turn then routes it out over the VPN tunnel).

### Source/Destination Checking

By default instances drop packets when source and destination information does not match that of an instance. This behavior can be disabled from the AWS console by selecting an instance and going to network options.

## Troubleshooting

This section contains some basic tunnel and routing troubleshooting commands.

### strongSwan Troubleshooting

You can run 'strongswan statusall' to view the status of the VPN tunnel. The expected output will look like below:

        $ strongswan statusall
        Status of IKE charon daemon (strongSwan 5.4.0, Linux 4.9.38-16.33.amzn1.x86_64, x86_64):
        uptime: 116 minutes, since Aug 04 12:46:06 2017
        malloc: sbrk 1622016, mmap 0, used 502608, free 1119408
        worker threads: 11 of 16 idle, 5/0/0/0 working, job queue: 0/0/0/0, scheduled: 2
        loaded plugins: charon aes des rc2 sha2 sha1 md4 md5 random nonce x509 revocation constraints acert pubkey pkcs1 pkcs8 pkcs12 pgp dnskey sshkey pem openssl gcrypt fips-prf gmp xcbc cmac hmac ctr ccm gcm curl attr kernel-netlink resolve socket-default farp stroke vici updown eap-identity eap-md5 eap-gtc eap-mschapv2 eap-tls eap-ttls eap-peap xauth-generic xauth-eap xauth-pam xauth-noauth dhcp
        Listening IP addresses:
        <strongSwan local address>
        <dummy address> ← both IP addresses should be listed
        Connections:
        iConectiv: <strongSwan local address>...<encrypted domain> IKEv1
        iConectiv: local: [<strongSwan pubilc address>] uses pre-shared key authentication
        iConectiv: remote: [<encrypted domain>] uses pre-shared key authentication
        iConectiv: child: <dummy address>/32 === <VPN Tunnel IP>/32 TUNNEL
        Security Associations (1 up, 0 connecting):
        iConectiv[1]: ESTABLISHED 116 minutes ago, <strongSwan local address>[<strongSwan public address>]...<encrypted domain>[<encrypted domain>]
        iConectiv[1]: IKEv1 SPIs: ce455418670b2bba_i* 09d884cb953d6c4d_r, pre-shared key reauthentication in 21 hours
        iConectiv[1]: IKE proposal: AES_CBC_128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
        iConectiv{3}: INSTALLED, TUNNEL, reqid 1, ESP SPIs: caab73dc_i 54015ba9_o
        iConectiv{3}: AES_CBC_128/HMAC_SHA1_96/MODP_1024, 1494 bytes_i (8 pkts, 2s ago), 611 bytes_o (8 pkts, 2s ago), rekeying in 47 minutes
        iConectiv{3}: <dummy address>/32 === <VPN Tunnel IP>/32

To find system messages related to strongSwan, use the following grep command:

    grep charon /var/log/messages

You can verify that the iptables rule to mask the source IP with the dummy address is properly configured. You should see the number of packets that have been applied to that rule:

    iptables -t nat -L -v

You can use the 'ip route' command to ensure strongSwan has properly configured the route on the instance required to forward traffic into the VPN tunnel. The second and fourth line of the below output is of particular importance:

    ip route show table all
