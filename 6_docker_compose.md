## Docker-Compose

**Refs:**
* [Overview of docker compose CLI](https://docs.docker.com/compose/reference/)

### Docker-Compose Example
```docker-compose -h```
_NOTE: make sure to be in the directory where the docker compose file is_

* Create a compose service ```docker-compose up -d```
[compose/commands/](/compose/commands/)
```bash
    [+] Running 9/9
     ✔ redis Pulled                                                            9.0s
       ✔ 24c63b8dcb66 Pull complete                                            6.2s
       ✔ 3fe6ac530baf Pull complete                                            6.2s
       ✔ 4c8c5b88360b Pull complete                                            6.2s
       ✔ a658802c95ef Pull complete                                            6.3s
       ✔ 1a120485a4ca Pull complete                                            6.5s
       ✔ 43e184461256 Pull complete                                            6.5s
       ✔ 4f4fb700ef54 Pull complete                                            6.5s
       ✔ 20ebfaaae36b Pull complete                                            6.6s
    [+] Running 4/4
     ✔ Network commands_default      Created                                   0.0s
     ✔ Volume "commands_nginx_html"  Created                                   0.0s
     ✔ Container commands-redis-1    Started                                   0.6s
     ✔ Container commands-web-1      Started                                   0.4s
```
* List running compose projects ```docker-compose ls```
* List containers created by compose ```docker-compose ps```
```bash
    NAME               IMAGE     COMMAND                  SERVICE   CREATED          STATUS          PORTS
    commands-redis-1   redis     "docker-entrypoint.s…"   redis     15 seconds ago   Up 14 seconds   6379/tcp
    commands-web-1     nginx     "/docker-entrypoint.…"   web       14 seconds ago   Up 13 seconds   0.0.0.0:8080->80/tcp
```
* Stopping a compose service ```docker-compose stop```
```bash
    [+] Stopping 2/2
     ✔ Container commands-web-1    Stopped                                     7.8s
     ✔ Container commands-redis-1  Stopped                                     0.1s
```
* Starting a compose service ```docker-compose start```
```bash
    [+] Running 2/2
     ✔ Container commands-redis-1  Started                                     0.1s
     ✔ Container commands-web-1    Started                                     0.1s
```
* Restarting a compose service ```docker-compose restart```
```bash
    [+] Restarting 2/2
     ✔ Container commands-redis-1  Started                                     0.2s
     ✔ Container commands-web-1    Started                                     0.2s
```
* Delete a compose service ```docker-compose down``` stop and remove containers, networks, images and volumes
```bash
    [+] Running 3/3
     ✔ Container commands-web-1    Removed                                     0.1s
     ✔ Container commands-redis-1  Removed                                     0.1s
     ✔ Network commands_default    Removed                                     0.1s
```

### Docker-Compose File
* [Compose specification](https://docs.docker.com/compose/compose-file/)
* [Docker Compose File](https://docs.docker.com/compose/compose-application-model/)
* Docker-compose file has 4 top level keys:
  * **[name](https://docs.docker.com/compose/compose-file/04-version-and-name/)**: define a project name
  * **[services](https://docs.docker.com/compose/compose-file/05-services/)**: define service definition, each of these definitions is going to represent a container that is built for the application.
    * if ```container_name``` is not specified, the name of the container will be the same as the service
    * if we change the name of the container in docker-compose file after we run it we need to re run (```docker-compose up -d```) it again so it recreates the application
    * [build](https://docs.docker.com/compose/compose-file/build/) option will build the container based on a provided ```context``` which is the path or directory where the docker file is located
      * if docker file has a custom name (not dockerfile) then the ```dockerfile``` option needs to be given.
      * if we make changes to our docker file, we need to re build it so the changes are taken ```docker-compose build --no-cache```
  * **[nerworks](https://docs.docker.com/compose/compose-file/06-networks/)**
  * **[volumes](https://docs.docker.com/compose/compose-file/07-volumes/)**
  * **[configs](https://docs.docker.com/compose/compose-file/08-configs/)**
  * **[secrets](https://docs.docker.com/compose/compose-file/09-secrets/)**