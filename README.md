![Docker Build Status](https://img.shields.io/docker/build/saponace/pia-client.svg)

# Open VPN client image for Private Internet Access

Create a an OpenVPN client container connected to a PrivateInternetAccess.com
server, specifically for use by other containers.

* Default route is removed from the container as a killswitch after the VPN
  tunnel is established.
* Health check is included so container can be restarted on loss of connectivity.
* Port forwarding is attempted and the forwarded port is stored in the named
  Docker volume `pia_port` as a text file also named `pia_port`.
* Can use any PIA region (endpoint)

## Build/Configure

* `docker build -t openvpn-client .`
* Create a file `pia-credentials` with PIA username on the
  first line and PIA password on the second. `chmod 600 pia-credentials`
* Copy `pia-credentials` to a local directory. This directory will be bind-mounted into the container as /config/credentials.
* Create a docker volume to share the forwarded port with other containers
  (without sharing the entire config directory). `docker volume create pia_port`

## Run

Note that because this container controls networking for other containers, _any
ports published that will be needed by other containers need to be published
when the openvpn container is started (unless a reverse proxy is being used)._

## Docker

### PIA client container
    docker run -d \
               --cap-add=NET_ADMIN \
               --device=/dev/net/tun \
               -v /pia-credentials:/config/credentials \
               -v /pia_port:/var/run/pia \
               -p 9091:9091 ## For Transmission container \
               --name=pia_client \
               pia-client

### Other container
    docker run -d \
               --net container:openvpn-client \
               -v /pia_port:/var/run/pia \
               --name=transmission_run \
               transmission


### Docker-compose
    # Other container
    transmission:
      image: transmission
      container_name: transmission
      networks:
          - vpn_client_net
      volumes:
        - /pia_port:/var/run/pia
    # PIA client container
    pia_client:
      image: pia-client
      container_name: pia_client
      restart: on-failure
      cap_add:
          - NET_ADMIN
      devices:
          - /dev/net/tun
      networks:
          - vpn_client_net
      volumes:
          - /pia-credentials:/config/credentials
          - /pia_port:/var/run/pia
      environment:
          - REGION=France


In these last two examples, `REGION` can be any region from https://www.privateinternetaccess.com/pages/network/

