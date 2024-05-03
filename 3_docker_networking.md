# Docker Networking

## Network Drivers
* **[bridge](https://docs.docker.com/network/drivers/bridge/)**: Default network driver, allows containers connected to the same bridge network to communicate, only works on linux. Commonly used when your application runs in a container that needs to communicate with other containers on the same host
* **[host](https://docs.docker.com/network/drivers/host/)**: Remove network isolation between the container and the Docker host, and use the host's networking directly
* **[overlay](https://docs.docker.com/network/drivers/overlay/)**: Connect multiple Docker daemons together and enable Swarm services and containers to communicate across nodes. This strategy removes the need to do OS-level routing.
* **[ipvlan](https://docs.docker.com/network/drivers/ipvlan/)**: IPvlan networks give users total control over both IPv4 and IPv6 addressing.
* **[macvlan](https://docs.docker.com/network/drivers/macvlan/)**: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network.
* **[none](https://docs.docker.com/network/drivers/none/)**: Completely isolate a container from the host and other containers.

*Ref: (https://docs.docker.com/network/drivers/)*

## General Commands
`docker network --help`
* **ls**: list all networks created
* **create**: creates a new network
  * **internal**: Restrict external access to the network, (i.e.) we are telling docker we dont want this to be bound to any of our interfaces
* **connect**: connect a container to a network
* **disconnect**: disconnect a container from a network
* **inspect**: inspects a network
* **prune**: deletes all unused network
* **rm**: deletes one or more network

## Working with Networks
1. Create a network `docker network create [nw_name]`
2. Connect a pre-existing container to a network `docker network connect [nw_name] [container_name]`
3. Disconnect a container from a network `docker network disconnect [nw_name] [container_name]`
4. Create a container and set the network `docker container run --network [nw_name] [image]`
   * Multiple networks `docker container run --network [nw_name1] --network [ns_name2] [image]`
5. Create a container and set the network and give an IP to it `docker container run --ip [x.x.x.x] --network [nw_name] [image]`
6. Create a network with subnet and gateway `docker network create --subnet [x.x.x.x/x] --gateway [x.x.x.x] [nw_name]`
7. Create a network with subnet and gateway and ip range
   ```sh
   docker network create \
   --subnet [x.x.x.x/x] \
   --gateway [x.x.x.x] \
   --ip-range=[x.x.x.x/x] \
   --driver=[driver_type] \
   --label=[label] \
   [nw_name]
   ```
   * Lets say we are going to have a parent network /16 and split it into smaller networks /24
      ```sh
      docker network create \
      --subnet 10.1.0.0/16 \
      --gateway 10.1.0.1 \
      --ip-range=10.1.1.0/24 \
      --driver=bridge \
      --label=network_1 \
      my_network
      ```
    -> Review why i cannot create other subnets aftewards
