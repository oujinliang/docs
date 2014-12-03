## IPSEC/L2TP VPN on Ubuntu 14.04 with OpenSwan, xl2tpd and ppp

### Follow the guide:
- https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_with_Ubuntu_14.04.html
- https://www.linode.com/stackscripts/view/2660


```
MyIp=`hostname -i`

apt-get install openswan xl2tpd ppp lsof

iptables -t nat -A POSTROUTING -j SNAT --to-source $MyIp -o eth0

echo "net.ipv4.ip_forward = 1" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.accept_redirects = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.send_redirects = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.rp_filter = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.accept_source_route = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.send_redirects = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.icmp_ignore_bogus_error_responses = 1" |  tee -a /etc/sysctl.conf

for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done

sysctl -p

```

Add these lines in `/etc/rc.local` 

```
MyIp=`hostname -i`
for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
iptables -t nat -A POSTROUTING -j SNAT --to-source $MyIp -o eth0
```

Edit `/etc/ipsec.conf`, replace %SERVERIP% with your server ip:

```
version 2 # conforms to second version of ipsec.conf specification

config setup
    dumpdir=/var/run/pluto/
    #in what directory should things started by setup (notably the Pluto daemon) be allowed to dump core?

    nat_traversal=yes
    #whether to accept/offer to support NAT (NAPT, also known as "IP Masqurade") workaround for IPsec

    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v6:fd00::/8,%v6:fe80::/10
    #contains the networks that are allowed as subnet= for the remote client. In other words, the address ranges that may live behind a NAT router through which a client connects.

    protostack=netkey
    #decide which protocol stack is going to be used.

    force_keepalive=yes
    keep_alive=60
    # Send a keep-alive packet every 60 seconds.

conn L2TP-PSK-noNAT
    authby=secret
    #shared secret. Use rsasig for certificates.

    pfs=no
    #Disable pfs

    auto=add
    #the ipsec tunnel should be started and routes created when the ipsec daemon itself starts.

    keyingtries=3
    #Only negotiate a conn. 3 times.

    ikelifetime=8h
    keylife=1h

    ike=aes256-sha1,aes128-sha1,3des-sha1
    phase2alg=aes256-sha1,aes128-sha1,3des-sha1
    # https://lists.openswan.org/pipermail/users/2014-April/022947.html
    # specifies the phase 1 encryption scheme, the hashing algorithm, and the diffie-hellman group. The modp1024 is for Diffie-Hellman 2. Why 'modp' instead of dh? DH2 is a 1028 bit encryption algorithm that modulo's a prime number, e.g. modp1028. See RFC 5114 for details or the wiki page on diffie hellmann, if interested.

    type=transport
    #because we use l2tp as tunnel protocol

    left=%SERVERIP%
    #fill in server IP above

    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any

    dpddelay=10
    # Dead Peer Dectection (RFC 3706) keepalives delay
    dpdtimeout=20
    #  length of time (in seconds) we will idle without hearing either an R_U_THERE poll from our peer, or an R_U_THERE_ACK reply.
    dpdaction=clear
    # When a DPD enabled peer is declared dead, what action should be taken. clear means the eroute and SA with both be cleared.
```
    
Edit `/etc/ipsec.secrets`: replace your server Ip, and generate random key `openssl rand -hex 20`

	%SERVERIP%  %any:   PSK "%RandomKey%"
	
Verify IPSEC Settings

	ipsec verify
	
Configure xl2tpd

Edit `/etc/xl2tpd/xl2tpd.conf`, replace the contents with the following:

```
[global]
ipsec saref = yes
saref refinfo = 30

;debug avp = yes
;debug network = yes
;debug state = yes
;debug tunnel = yes

[lns default]
ip range = 172.16.1.30-172.16.1.100
local ip = 172.16.1.1
refuse pap = yes
require authentication = yes
;ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

Configuring PPP

Edit `/etc/ppp/options.xl2tpd` , replace the content with:

```
require-mschap-v2
ms-dns 8.8.8.8
ms-dns 8.8.4.4
auth
mtu 1200
mru 1000
crtscts
hide-password
modem
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
```

Adding users

Every user should be defined in the `/etc/ppp/chap-secrets` file. Below is an example file:

```
# Secrets for authentication using CHAP
# client       server  secret                  IP addresses
alice          l2tpd   0F92E5FC2414101EA            *
bob            l2tpd   DF98F09F74C06A2F             *
```

Testing it

```
/etc/init.d/ipsec restart 
/etc/init.d/xl2tpd restart
```

On the client connect to the server IP address (or add a DNS name) with a valid user, password and the shared secret.