## Docker Storage

## Notes
* Default volume location in linux is `/var/lib/docker/volumes/[volume_name]/_data`
* Volumes can be attached to multiple containers

## General Commands
`docker volume --help`
* **create**: creates a volume
* **inspect**: display detailed information on one or more volumes
* **ls**: list volumes
* **prune**: remove unused local volumes
* **rm**: remove one or more volumes

## Bind Mounts
1. Attach a Bind Mount using mount flag to a container `docker container run -d --mount type=bind,source=[host_source_path],target=[container_target_path] [image]`
   * This requires that the source path is already created in the Host, example:
        `
        mkdir my_directory
        docker container run -d --mount type=bind,source="$(pwd)"/my_directory,target=/app nginx
        `
    * Bind mounts are not managed by docker volumes therefore `docker volume ls` will not show anything
2. Attach a Bind Mount using volume flag to a container `docker container run -d -v [host_source_path]:[container_target_path] [image]`
   * Does not require to have the directory created beforehand `docker container run -d -v "$(pwd)"/new_directory:/app nginx`
   * Bind only a file `docker container run -d -v "$(pwd)"/nginx/nginx.conf:/etc/nginx/nginx.conf nginx`

## Volumes for Persistent Storage
1. Create volume `docker volume create [vol_name]`
2. Create a container and mount a volume with mount flag `docker container run --mount type=volume,source=[host_source_path],target=[container_target_path] [image]`
3. Create a container and mount a volume with mount flag and readonly `docker container run --mount type=volume,source=[host_source_path],target=[container_target_path],readonly [image]`
4. Create a container and mount a volume with volume flag `docker container run -v [vol_name]:[container_target_path] [image]`