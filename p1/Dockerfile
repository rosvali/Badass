FROM frrouting/frr

# RUN docker exec -it . sh
RUN sed -i -e "s/bgpd=no/bgpd=yes/g" /etc/frr/daemons
    # sed -i -e "s/ospfd=no/ospfd=yes/g" /etc/frr/daemons &&
    # sed -i -e "s/isisd=no/isisd=yes/g" /etc/frr/daemons