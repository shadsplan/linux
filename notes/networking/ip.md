# ip

``` bash

# Go-to command to view information regarding network interfaces on our system.
shad@linux:~/linux/notes/networking$ ip addr show # can also do "ip a"

# lo = loopback interface. It's how processes on your machine talk to each other. (ex. curl localhost).
# 127.0.0.1 is the loopback address.
# We have a max packet size (mtu) of 65536 bytes on lo.
# UP means the interface is enabled. LOWER_UP means the physical layer is up (cable plugged in, etc).
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever

# eth0 = ethernet interface 0. This is our primary network interface. How it talks to other machines on the network (Azure VNET).
# It has a MAC address of 60:45:bd:ef:f3:4d. Layer 2 identifier.
# It has an IPv4 address of 10.0.0.4/24. The /24 means the subnet mask is 255.255.255.0.
# brd 10.0.0.255 is the broadcast address.
# We have a max packet size (mtu) of 1500 bytes on eth0.
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 60:45:bd:ef:f3:4d brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6245:bdff:feef:f34d/64 scope link 
       valid_lft forever preferred_lft forever

# enP14966s1 = ethernet interface 1. This is a virtual interface, a sl*ve to eth0.
3: enP14966s1: <BROADCAST,MULTICAST,SL*VE,UP,LOWER_UP> mtu 1500 qdisc mq master eth0 state UP group default qlen 1000
    link/ether 60:45:bd:ef:f3:4d brd ff:ff:ff:ff:ff:ff
    altname enP14966p0s2
    inet6 fe80::6245:bdff:feef:f34d/64 scope link 
       valid_lft forever preferred_lft forever

# View the routing table on our system.
#  
shad@linux:~$ ip route show

# default gateway. Access something via network, it will go through this route.
# if we remove this, we can't access the internet.
default via 10.0.0.1 dev eth0 proto dhcp src 10.0.0.4 metric 100

10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.4 metric 100 
10.0.0.1 dev eth0 proto dhcp scope link src 10.0.0.4 metric 100 
168.63.129.16 via 10.0.0.1 dev eth0 proto dhcp src 10.0.0.4 metric 100 
169.254.169.254 via 10.0.0.1 dev eth0 proto dhcp src 10.0.0.4 metric 100 
```