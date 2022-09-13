# Badass

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
