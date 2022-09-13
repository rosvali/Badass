# Badass

- [p1](#p1)
- [p2](#p2)

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

> VXLAN is an overlay network and carry Ethernet traffic between 2 machines

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
