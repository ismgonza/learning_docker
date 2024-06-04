## Docker Security

**Refs:**
* [Docker Bench Security](https://github.com/docker/docker-bench-security)
* [Resource Constrains](https://docs.docker.com/config/containers/resource_constraints/)
* [Runtime privilege and Linux capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)
* 


### Docker Namespaces
* Process ID (pid)
* Network (net)
* Fylesystem/mount (mount)
* Inter-process Communication (ipc)
* User (user)
* UTS (uts)

### Docker Secrets
* Only available in Swarm mode
* Secrets are encrypted and store
* Secrets are encrypted in-flight

### Seccomp Profiles
* Testing Seccomp:
```bash
    docker container run --rm -it alpine sh
    whoami
    mount /dev/sda1 /tmp
    swapoff -a
```
* Using a custom seccomp profile
```bash
    docker container run --security-opt seccomp=[PROFILE] [IMAGE] [CMD]
```
* Actual Example:
```bash
    mkdir -p seccomp/profiles/chmod
    cd seccomp/profiles/chmod
    wget https://raw.githubusercontent.com/moby/moby/master/profiles/seccomp/default.json
    # Remove chmod, fchmod and fchmodat from the syscalls whitelist. Syscalls starts at line 52.
```
* Applying the custom Seccomp profile:
```bash
    docker container run --rm -it --security-opt seccomp=./default.json alpine sh
    chmod +r /usr
```
### Capabilities:
* Adding Capabilities: `docker container run --cap-add=[CAPABILITY] [IMAGE] [CMD]`
* Dropping Capabilities: `docker container run --cap-drop=[CAPABILITY] [IMAGE] [CMD]`
* [Runtime privilege and Linux capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)

### Control Groups
* Limiting CPU and memory:
`docker container run -it --cpus=[VALUE] --memory=[VALUE][SIZE] --memory-swap [VALUE][SIZE] [IMAGE] [CMD]`
  * Setting memory limits on a container:
    `docker container run -d --name resource-limits --cpus=".5" --memory=512M --memory-swap=1G rivethead42/weather-app`
  * Inspect resource-limits:
    `docker container inspect resource-limits`
  * [Runtime constraints on resources](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)
  * [More info on resource constraints](https://docs.docker.com/config/containers/resource_constraints/)

### Running Docker Bench for Security
* Running Docker Bench Security:
``` bash
    docker container run --rm -it --network host --pid host --userns host --cap-add audit_control \
        -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
        -v /var/lib:/var/lib \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v /usr/lib/systemd:/usr/lib/systemd \
        -v /etc:/etc --label docker_bench_security \
        docker/docker-bench-security
```

## Docker content trust
* Creating a Key `docker trust key generate [NAME]`
* Importing a Key `docker trust key load [PEM] --name [NAME]`
* Creating a Key `docker trust signer add --key [PEM] [NAME] [REPOSITORY]`
* Creating a Key `docker trust signer remove [NAME] [REPOSITORY]`
* Signing an image `docker trust sign [IMAGE]:[TAG]`
* Enabling DCT `vi /etc/docker/daemon.json`
```json
    {
        "content-trust": {
            "mode": "enforced"
        }
    }
```

## Docker Secrets
* Creating a secret using STDIN `STDIN | docker secret create [NAME] -`
  * `openssl rand -base64 20 | docker secret create my_secret_data -`
* Creating a secret using a file `docker secret created [NAME] [FILE]`
* List secrets `docker secret ls`
* Inspecting a secret `docker secret inspect [NAME]`
* Using a secret `docker service create --name [NAME] --secret [SECRET] [IMAGE]`
  * See the content of the secrets `docker container exec $(docker ps --filter name=[NAME] -q) cat /run/secrets/my_secret_data`
* Delete a secret `docker secret rm [NAME]`

### Real life example with WP blog
* Generate password files:
```bash
    openssl rand -base64 20 > db_password.txt
    openssl rand -base64 20 > db_root_password.txt
```
* Create a Wordpress Stack: `vi docker-compose.yml`
* docker-compose.yml contents:
  * Important to keep in mind that docker will look for environment keys that end in _FILE, [check line #30](https://github.com/docker-library/mysql/blob/master/8.0/docker-entrypoint.sh)
  * Since secrets are mounted in memory we need to provide this path `/run/secrets/[NAME OF THE SECRET]`
```dockerfile
    services:
       db:
         image: mysql:5.7
         volumes:
           - db_data:/var/lib/mysql
         networks:
           mysql_internal:
             aliases: ["db"]
         environment:
           MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
           MYSQL_DATABASE: wordpress
           MYSQL_USER: wordpress
           MYSQL_PASSWORD_FILE: /run/secrets/db_password
         secrets:
           - db_root_password
           - db_password

       wordpress:
         depends_on:
           - db
         image: wordpress:latest
         networks:
           mysql_internal:
             aliases: ["wordpress"]
           wordpress_public:
         ports:
           - "8001:80"
         environment:
           WORDPRESS_DB_HOST: db:3306
           WORDPRESS_DB_USER: wordpress
           WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
         secrets:
           - db_password

    secrets:
       db_password:
         file: db_password.txt
       db_root_password:
         file: db_root_password.txt

    volumes:
        db_data:
    networks:
      mysql_internal:
        driver: "overlay"
        internal: true
      wordpress_public:
        driver: "overlay"
```
* Deploy stack `docker stack deploy --compose-file docker-compose.yml wp`