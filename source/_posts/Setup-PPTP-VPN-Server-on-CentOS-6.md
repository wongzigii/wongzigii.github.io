title: Setup PPTP VPN Server on CentOS 6
date: 2016-03-29 16:57:00
tags: VPS, CentOS
---

This article will not dive into the installation of pptpd. Instead, I would like to blog down the configurations and some issues I came cross with the follow-up work.

So here, you've just installed the pptpd through `yum install` on CentOS, or `apt-get install` on Ubuntu.

CentOS version: centos-release-6-7.el6.centos.12.3.x86_64

## 1. Edit IP settings

````bash
vi /etc/pptpd.conf
````
    
Uncomment `localip` and `remoteip` lines and replace the following IP with you local IP address and remote IP address.

````bash
localip 192.168.9.1
remoteip 192.168.9.11-30
````
The localip is the local IP address of server, while remoteip, as its name implies, the *range* of remote IPs being able to distribution.
 
## 2. Configure Usernames and Passwords

````bash
vi /etc/ppp/chap-secrets
````

Change the username and password accordingly.

````bash
username1  pptpd   Pp$$w0rd  *
username2  pptpd   P@$$w0rd2  *
````

Note that there is `*` at the end of the line.

## 3. Enable ipv4 network forwarding

````bash
vi /etc/sysctl.conf
````

> net.ipv4.ip_forward = 1

And then apply the change with the following command.

````bash
sysctl -p
````
    
(Re)start service

````bash
service pptpd start
````

So far so good. Now you're able to connect to VPN server.

# Troubleshoting

However, I could not access the network resources over the VPN server.

So far I've confirmed:

- ping successfully on the server side so rule DNS issue out as the cause of the problem.
- ping to server side sucessfully from client side.
- use `ifconfig` to check out `ppp0` interface is working.

So, it sounds like the default route is not being set correctly when VPN connection is brought up.

## Configure routing with iptables

We're going to use this following command to check out the `eth0(1)` or `seth0` interface. 

````bash
ifconfig
````
The VPN server is listening for PPTP traffic on TCP port 1723 and port 1701 for L2TP traffic on UDP. So apply this rule to iptables:

````bash
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 1723 -j ACCEPT
````
    
Also, apply `NAT` rule. 

````bash
iptables -t nat -A POSTROUTING -o seth0 -s 192.168.9.0/24 -j SNAT --to-source 118.193.160.45
````
    
`192.168.9.0/24` means `192.168.9.0 255.255.255.0`, which `192.168.9.0` is the incoming address, as `remoteip` we've early configured in `pptpd.conf`, and the part after the slash, in this case, `24`, is how many subnet mask bits to use. So `255.255.255.0` is using 24 of the 32 bits to create the subnet.
At the same time, `118.193.160.45` is your server's public IP address.

You're almost there! Next step, save iptables you've added
 
````bash
service iptables save
````
 
Finally, restart iptables.

````bash
service iptables start
````

Done!
