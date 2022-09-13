FROM frrouting/frr

# active protocol useful for the project
RUN sed -i -e "s/bgpd=no/bgpd=yes/g" /etc/frr/daemons
RUN sed -i -e "s/ospfd=no/ospfd=yes/g" /etc/frr/daemons
RUN sed -i -e "s/isisd=no/isisd=yes/g" /etc/frr/daemons