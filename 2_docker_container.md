# Working with Containers

## General Commands
`docker container --help`
* **ls**: list all running containers (add -a to list stopped containers too)
* **run**: create and run a new container (different from create which only creates but doesnt run)
* **attach**: attaches to a running container (attaches to local standard input, output, and error streams to a running container)
* **exec**: executes a command in a running container
* **start**: starts a stopped container
* **stop**: stops a running container
* **inspect**: provides details about a container (network, volmes, etc)
* **rm**: deletes a stopped container
* **prune**: deletes all stopped containers
* **top**: display running processes of a container
* **logs**: show logs from the container
* **rename**: rename a container

## Creating Containers (simple)
1. Create and run a container `docker container run [image]`
2. Create and run a container and delete it when exited `docker container run --rm [image]`
3. Create and run a container with a custom name `docker container run --name [container_custom_name] [image]`
4. Create and run a container and detach (run in the background) `docker container run -d [image]`
5. Create and run a container and keep terminal open `docker container run -it [image]`

## Exposing and Publishing Ports for Containers
1. Exposing/opening a port on a container `docker container run -d --expose 3000 nginx`
2. Mapping it to the container `docker container run -p 8080: 80 nginx`
   * 8080: 80 is the port 8080 on the host mapped to 80 on the container, this applies for UDP and TCP
   * if we want only TCP or UDP we need to specify it like this `docker container run -p 8080: 80/tcp nginx`
   * if we want multiple ports and protocols, we can do `docker container run -p 8080: 80/tcp -p 8080: 80/udp nginx`
3. List all ports for container `docker container port [container_name]`

## Executing Container Commands
1. Create and run a container with a command `docker container run [image] [cmd]`
   * Run bash `docker container run -it nginx /bin/bash`
2. Just execute a command `docker container exec -it [container_name] ls /usr/share/nginx/html`

## Delete and Prine Containers
1. Delete container `docker container rm [container_name] -f`
2. Delete containers `docker container rm [container_name1] [container_name2] -f`
3. Prune containers `docker container prune`

## Advance Managing Containers
### Inspecting Container Processes
* Docker Top: display running processes of a container
  `docker container top [container_name]`
* Docker Stats: Display lifestream of container resource usage statistics
  `docker container stats [container_name]`

### Auto-Restart Containers
* To autorestart use the `--restart` flag when creating the container
* Set one of the following options:
  * `--restart no`: is the default, do not restart automatically
  * `--restart on-failure`: restart if container exists due to a failure (non-zero exit code)
  * `--restart always`: always restart the container if it stops and then the docker daemon is restarted
  * `--restart unless-stopped`: similar to always except it does not restart the container if it is intentionally stopped
* Syntax: `docker container run -d --name [container_name] --restart [restart_option] [image]`

### Docker Events
* Get real-time events from the server
  `docker system events`
  `docker system events --since '[time-period]'`
* Generate Events:
  `docker container exec docker_events /bin/bash`
  `docker container attach docker_events`
  `docker container start docker_events`
* Filters Events:
  `docker system events --filter <FILTER_NAME>=<FILTER>`
* Filter for container events:
  `docker system events --filter type=container --since '1h'`
  `docker system events --filter type=container --filter event=start --since '1h'`
* Filter for attach events:
  `docker system events --filter type=container --filter event=attach`
* Connect to docker_events using /bin/bash:
  `docker container exec -it docker_events /bin/bash`
* Use multiple filters:
  `docker system events --filter type=container --filter event=attach --filter event=die --filter event=stop`
* Documentation:
   * [docker events](https://docs.docker.com/reference/cli/docker/system/events/)
   * [Engine API v1.24](https://docs.docker.com/engine/api/v1.24/)

### Managing stopped containers
* List the rm flags:
  `docker container rm -h`
* List the IDs of all containers:
  `docker container ls -a -q`
* List all stopped containers:
  `docker container ls -a -f status=exited`
* List the IDs of stopped containers:
  `docker container ls -a -q -f status=exited`
* Get a count of all stopped containers:
  `docker container ls -a -q -f status=exited | wc -l`
* Fin stopped weather-app containers with grep:
  `docker container ls -a -f status=exited | grep weather-app`

### Docker Portainer
* In this lesson, we'll install Portainer and use it manage our Docker host.
* Create a volume for Portainers data:
  `docker volume create portainer_data`
* Create the Portainers container:
  ```sh
  docker container run -d --name portainer -p 8080:9000 \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer
  ```
* Access the portainer by using localhost:8080

### Watchtower
* To keep a container up-to-date when its image gets updated
* Create Watchtower:
  ```sh
  docker container run -d --name watchtower \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower -i 15
  ```