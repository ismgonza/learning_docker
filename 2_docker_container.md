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