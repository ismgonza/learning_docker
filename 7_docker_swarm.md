## Docker Swarm

**Refs:**
* [Docker Swarm Overview](https://docs.docker.com/engine/swarm/)

### Docker Swarm Structure
* Swarm has two major components:
  * Enterprise Cluster
  * Orchestration Enginer

### An enterprise grade secure cluster:
* Manage one or more Docker nodes as a cluster
* Encrypted distributed cluster store
* Encrypted networks
* Secure join tokens

### An orchestration engine for creating mircoservices:
* API for deploying and managing microservices
* Declarative manifest files for defining apps
* Provides availability to scale apps, and perform rolling updates and rollbacks

_Note: Swarm was initially a separate product layered on Docker, since Docker 1.12 it has become a part of the engine._

### The Cluster
* A swarm consists of one or more Docker nodes.
* Nodes are either a managers or a worker.
  1. Managers:
    * Manage the state of the cluster
    * Dispatch tasks to workers
  2. Workers:
    * Accepts and execute tasks
* Configuration data and State of the Swarm is held in etcd
* Swarm uses Transport Layer Security (TLS):
  * Encrypted communication
  * Authenticated nodes
  * Authorized roles
  * Orchestration
* The atomic unit of scheduling is a swarm service.
* The service construct adds the following to a container:
  * scaling
  * rolling updates
  * rollback
  * updates
* A container wrapped in a service is a task or a replica.

### Commands
* Install docker swarm
* On Master, run
```docker swarm init --advertise-addr [Private IP of the Swarm Master]```
* Copy the `docker swarm join` command provided (to be used on the workers after they have Docker setup.)
* On workers, run:
* `docker swarm join`
* Example: `docker swarm join --token SWMTKN-1-058yz85mc76fsf1ldm7w2evqjuit5nsdksrzqbtgtnaetnze1m-4w76mpkecb5y1utjexhdhlly8 172.31.37.43:2377`
* After completing the install go back to Master and run:
* `docker node ls`

### Managing the Swarm
* Listing Nodes `docker node ls`
* Inspecting a node `docker node inspect [NODE_NAME]`
* Promoting Worker to be a manager `docker node promote [NODE_NAME]`
* Demoting a manager to worker `docker node demote [NODE_NAME]`
* Removing a node from the swarm `docker node rm -f [NODE_NAME]`
* Have a node leave the swarm `docker swarm leave` on the Manager
  * If a node is removed from the swarm and we want to add it back in we need to run `leave` otherwise the node will think it is still part of the swarm and will generate errors
* If we lost the join tocken we can get it again by running `docker swarm join-token` in the Manager
* Generate a token to add a new node as a Manager `docker searm join-token manager`, and run it on the new node

### Working with Services
* Creating a Service: `docker service create -d --name [NAME] -p [Host_Port]:[Container_port] --replicas [#_Replicas] [Image] [CMD]`
* List services `docker service ls`
* Inspect services `docker service inspect [NAME]`
  * Check for the Network the service is attached to `docker network ls --no-trunk | grep [NetworkID]`
* Getting logs for a service `docker service logs [NAME]`
* List all tasks for a service `docker service ps [NAME]`
* Scaling up and down a service `docker service scale [NAME]=[#_Replicas]`
* Updating a service `docker service update [OPTIONS] [NAME]`

### Networking in Docker Swarm
* Create [overlay network](/3_docker_networking.md) `docker network create -d -overlay [NAME]`
* Creating a service with an overlay network `docker service create -d --name [NAME] --network [NETWORK] -p [Host_port]:[Container_port] --replicas [#_Replicas] [IMAGE] [CMD]`
* Adding a service to a network `docker service update --network-add [NETWORK] [SERVICE]`
* Remove a service from a network `docker service update --network-rm [NETWORK] [SERVICE]`

### Extending Docker with Plugins
* Install Plugins `docker plugin install [PLUGIN] [OPTIONS]`
* Listing Plugins `docker plugin ls`
* Disable plugin `docker plugin disable [ID]` (required before removing it)
* Remove plugin `docker plugin rm [ID]`
* [Docker Plugins](https://docs.docker.com/engine/extend/)

### Volumes with Docker Swarm
* Create a volume using a driver
`docker volume create -d [DRIVER] [NAME]`
```bash
    docker service create -d --name [NAME] \
    --mount type=[TYPE],src=[SOURCE],dst=[DESTINATION] \
    -p [HOST_PORT]:[CONTAINER_PORT] \
    --replicas [REPLICAS] \
    [IMAGE] [CMD]
```
* Create a volume on the manager: `docker volume create -d local portainer_data`
* Create a portainers service that uses a volume and run locally on the manager node (constrained):
```bash
    docker service create \
    --name portainer \
    --publish 8000:9000 \
    --constraint 'node.role == manager' \
    --mount type=volume,src=portainer_data,dst=/data \
    --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    portainer/portainer \
    -H unix:///var/run/docker.sock
```
  * `--constraint`: https://docs.docker.com/reference/cli/docker/service/create/#constraint

### [Deploying Stacks in Docker Swarm](https://docs.docker.com/engine/swarm/stack-deploy/)
* Docker Stack is run across a Docker Swarm, which is essentially a group of machines running the Docker daemon, which are grouped together, essentially pooling resources. Stacks allow for multiple services, which are containers distributed across a swarm, to be deployed and grouped logically.
* Lets make an example with prometheus:
  1. Set up the environment (create a swarm/prometheus directory)
  2. Create a [prometheus.yml](/swarm/prometheus/prometheus.yml) file
  3. Create a [docker-compose.yml](/swarm/prometheus/docker-compose.yml) file
  4. Deploy the Stack `docker stack deploy --compose-file docker-compose.yml prometheus`
  5. List stacks `docker stack ls`
  6. List services `docker service ls`
  7. Fix volume permissions `sudo chown nfsnobody:nfsnobody -R /var/lib/docker/volumes/prometheus_data`
  8. Delete a Stack `docker stack rm [NAME]`

