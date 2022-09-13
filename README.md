# Badass

- [p1](#p1)
- [p2](#p2)
- [p3](#p3)

# p1

## Create two docker images
```
sh /home/vagrant/Desktop/badass/build.sh
```

build.sh
```
# pull alpine image directly from dockerhub
docker pull alpine:latest

# build an image from a dockerfile and named it frrouting/frr
docker build -t frrouting/frr /home/vagrant/Desktop/badass/p1
```

Dockerfile
```
FROM frrouting/frr

# active protocol useful for the project
RUN sed -i -e "s/bgpd=no/bgpd=yes/g" /etc/frr/daemons
RUN sed -i -e "s/ospfd=no/ospfd=yes/g" /etc/frr/daemons
RUN sed -i -e "s/isisd=no/isisd=yes/g" /etc/frr/daemons
```

## Display configuration

Import `P1.gns3project`

Display processus and hostname in each machines
```
ps
hostname
```

# p2

> VXLAN is an overlay network and carry Ethernet traffic between machines

[Doc VXLAN](https://vincent.bernat.ch/fr/blog/2017-vxlan-linux)

Configure routers
```
# create and set a bridge
ip link add br0 type bridge
ip link set dev br0 up
```

## Configure VXLAN in static

in router 1
```
ip addr add 10.1.1.1/24 dev eth0
# create and configure a vxlan
ip link add name vxlan10 type vxlan id 10 dev eth0 remote 10.1.1.2 local 10.1.1.1 dstport 4789
ip addr add 20.1.1.1/24 dev vxlan10
```

in router 2
```
ip addr add 10.1.1.2/24 dev eth0

ip link add name vxlan10 type vxlan id 10 dev eth0 remote 10.1.1.1 local 10.1.1.2 dstport 4789
ip addr add 20.1.1.2/24 dev vxlan10
```

## Configure VXLAN in multicast

in router 1
```
ip addr add 10.1.1.1/24 dev eth0

ip link add name vxlan10 type vxlan id 10 dev eth0 group 239.1.1.1  dstport 4789
ip addr add 20.1.1.1/24 dev vxlan10
```

in router 2
```
ip addr add 10.1.1.2/24 dev eth0

# multicast is configured by specified a group
ip link add name vxlan10 type vxlan id 10 dev eth0 group 239.1.1.1  dstport 4789
ip addr add 20.1.1.2/24 dev vxlan10
```

----
```
# make the interface a port of the bridge
brctl addif br0 eth1 
brctl addif br0 vxlan10
ip link set dev vxlan10 up
```

Configure hosts
```
# in host 1
ip addr add 30.1.1.1/24 dev eth1

ping 30.1.1.2

# in host 2
ip addr add 30.1.1.2/24 dev eth1

ping 30.1.1.1
```

Then start a capture with Wireshark

# p3

> BGP (Border Gateway Protocol) a protocol designed to exchange routing and reachability information among autonomous systems

> EVPN Ethernet VPN enables you to connect dispersed customer sites using a layer 2 virtual bridge

> OSPF (open shortest path first) is a protocol that choose the best path for packages and traffic to take between connected networks

> layer 2 allows host devices in the same subnet to send bridged or Layer 2 traffic to each other

# Set leaf switchs

router 1
```
hostname raguillo-1
no ipv6 forwarding
!
interface eth0
 ip address 10.1.1.1/30
!
interface eth1
 ip address 10.1.1.5/30
!
interface eth2
 ip address 10.1.1.9/30
!
interface lo
 ip address 1.1.1.1/32
!
router bgp 1
# create a bgp group
 neighbor ibgp peer-group
 # se the as number to 1
 neighbor ibgp remote-as 1
 # for bgp to use lo when talking to a neighbor
 neighbor ibgp update-source lo
 bgp listen range 1.1.1.0/29 peer-group ibgp
 !
 address-family l2vpn evpn
 # active ibgp to exchange info with neighbor
  neighbor ibgp activate
  # identified the router as a rr
  neighbor ibgp route-reflector-client
 exit-address-family
!
router ospf
 network 0.0.0.0/0 area 0
!
line vty
!
```

router 2
```
# add a vxlan configuration
ip link add br0 type bridge
ip link set dev br0 up
ip link add vxlan10 type vxlan id 10 dstport 4789
ip link set dev vxlan10 up
brctl addif br0 vxlan10
brctl addif br0 eth1

hostname raguillo-2
no ipv6 forwarding
!
interface eth0
 ip address 10.1.1.2/30
 ip ospf area 0
!
# loopback est l'interface réseau réservée utilisée par le système local pour permettre les communications entre processus
interface lo
 ip address 1.1.1.2/32
 ip ospf area 0
!
router bgp 1
 neighbor 1.1.1.1 remote-as 1
 neighbor 1.1.1.1 update-source lo
 !
 address-family l2vpn evpn
  neighbor 1.1.1.1 activate
  advertise-all-vni
 exit-address-family
!
router ospf
!
```

router 3
```
hostname raguillo-3
no ipv6 forwarding
!
interface eth1
 ip address 10.1.1.6/30
 ip ospf area 0
!
interface lo
 ip address 1.1.1.3/32
 ip ospf area 0
!
router bgp 1
 neighbor 1.1.1.1 remote-as 1
 neighbor 1.1.1.1 update-source lo
 !
 address-family l2vpn evpn
  neighbor 1.1.1.1 activate
 exit-address-family
!
router ospf

```

# Set router reflector

router 4
```
hostname raguillo-1
no ipv6 forwarding
!
interface eth0
 ip address 10.1.1.1/30
!
interface eth1
 ip address 10.1.1.5/30
!
interface eth2
 ip address 10.1.1.9/30
!
interface lo
 ip address 1.1.1.1/32
!
router bgp 1
 neighbor ibgp peer-group
 neighbor ibgp remote-as 1
 neighbor ibgp update-source lo
 bgp listen range 1.1.1.0/29 peer-group ibgp
 !
 address-family l2vpn evpn
  neighbor ibgp activate
  neighbor ibgp route-reflector-client
 exit-address-family
!
router ospf
 network 0.0.0.0/0 area 0
!
line vty
!
```

# Test

```
do sh ip route
do sh bgp summary
```

show route before host configuration
```
do sh bgp l2vpn evpn 
```

host 1
```
ip addr add 20.1.1.1 dev eth1
```

show route after host configuration
```
do sh bgp l2vpn evpn 
```

host 3
```
ip addr add 20.1.1.3 dev eth0
```

in host 1
```
ping 20.1.1.3
```