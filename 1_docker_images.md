# Working with Images

## General Commands
`docker image --help`
* **ls**: list all available, or pulled images
* **pull**: pulls or downloads a docker image from a registry
* **push**: pushes or uploads a docker image to a registry
* **inspect**: provides details on the docker image
* **import**: imports content from a tarball to create a filesystem image
* **rm / rmi**: deletes a specific image
* **prune**: deletes all unused images

# Building and Distributing Images
`docker image build -t [name]:[tag] .`
* **`-f, --file string`**: name of the docker file (if different than dockerfile)
* **`--force-rm`**: always remove intermediate containers
* **`--label`**: sets metadata for an image
* **`--rm`**: removes intermediate container after a successful build
* **`--ulimit`**: ulimit options

### Other options
`docker image build -t [name]:[tag] [git_url]#[branch]`
`docker image build -t [name]:[tag] [git_url]#:[directory_where_dockerfile_is]`
`docker image build -t [name]:[tag] [git_url]#[branch]:[directory_where_dockerfile_is]`
`docker image build -t [name]:[tag] - < [file].tar.gz`

## Multi-stage builds
* Stages are not named
```dockerfile
    FROM node AS build
    RUN mkdir -p /var/node/
    ADD src/ /var/node/
    WORKDIR /var/node
    RUN npm install

    FROM node:alpine
    ARG VERSION=V1.1
    LABEL org.label-schema.version=$VERSION
    ENV NODE_ENV="production"
    COPY --from=build /var/node /var/node
    WORKDIR /var/node
    EXPOSE 3000
    ENTRYPOINT ["./bin/www"]
```
* Build the image:
```sh
    docker image build -t linuxacademy/weather-app:multi-stage-build \
    --rm \
    --build-arg VERSION=1.5 .
```
## Using Tags
* Add a name and an optional tag with -t or --tag, in the name:tag format:
```sh
    docker image build -t <name>:<tag>
    docker image build --tag <name>:<tag>
```
* List your images:
  `docker image ls`
* Use our Git commit hash as the image tag:
  `git log -1 --pretty=%H`
* Use the Docker tag to a create a new tagged image:
  `docker tag <SOURCE_IMAGE><:TAG> <TARGET_IMAGE>:<TAG>`
* Get the commit hash:
```sh
    cd docker_images/weather-app/src
    git log -1 --pretty=%H
    cd ../
```
* Build the image using the Git hash as the tag:
  `docker image build -t linuxacademy/weather-app:<GIT_HASH> .`
* Tag the weather-app as the latest using the image tagged with the commit hash:
  `docker image tag linuxacademy/weather-app:<GIT_HASH> linuxacademy/weather-app:latest`

## Pushing to Dockerhub
* Docker login
  `docker login`
* Docker Push:
  `docker image push <USERNAME>/<IMAGE_NAME>:<TAG>`


# Managing Images
* Docker image history
  `docker image history [image]`
  Example: (reads from bottom to top)
  First get the list of images
  ```sh
  $ docker image ls
  REPOSITORY             TAG       IMAGE ID       CREATED       SIZE
  ismgonza/weather-app   latest    67a1056ced8b   5 days ago    164MB
  nginx                  latest    a6ac09e4d8a9   3 weeks ago   193MB
  mysql                  latest    9e24fab8e6af   6 weeks ago   638MB
  centos                 latest    e6a0117ec169   2 years ago   272MB
  ```
  Now lets get the image history
  ```sh
  $ docker image history ismgonza/weather-app
  IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
  67a1056ced8b   5 days ago     ENTRYPOINT ["./bin/www"]                        0B        buildkit.dockerfile.v0
  <missing>      5 days ago     EXPOSE map[3000/tcp:{}]                         0B        buildkit.dockerfile.v0
  <missing>      5 days ago     WORKDIR /node/weather-app                       0B        buildkit.dockerfile.v0
  <missing>      5 days ago     COPY /node/weather-app /node/weather-app # b…   16.5MB    buildkit.dockerfile.v0
  <missing>      5 days ago     ENV NODE_ENV=production                         0B        buildkit.dockerfile.v0
  <missing>      5 days ago     LABEL org.label-schema.version=V1.1             0B        buildkit.dockerfile.v0
  <missing>      5 days ago     ARG APP_VERSION=V1.1                            0B        buildkit.dockerfile.v0
  <missing>      7 days ago     /bin/sh -c #(nop)  CMD ["node"]                 0B
  <missing>      7 days ago     /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
  <missing>      7 days ago     /bin/sh -c #(nop) COPY file:4d192565a7220e13…   388B
  <missing>      7 days ago     /bin/sh -c apk add --no-cache --virtual .bui…   5.57MB
  <missing>      7 days ago     /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.19     0B
  <missing>      7 days ago     /bin/sh -c addgroup -g 1000 node     && addu…   135MB
  <missing>      7 days ago     /bin/sh -c #(nop)  ENV NODE_VERSION=22.1.0      0B
  <missing>      3 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
  <missing>      3 months ago   /bin/sh -c #(nop) ADD file:37a76ec18f9887751…   7.38MB
  ```
  Following part is the base image
  ```sh
  <missing>      7 days ago     /bin/sh -c #(nop)  CMD ["node"]                 0B
  <missing>      7 days ago     /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
  <missing>      7 days ago     /bin/sh -c #(nop) COPY file:4d192565a7220e13…   388B
  <missing>      7 days ago     /bin/sh -c apk add --no-cache --virtual .bui…   5.57MB
  <missing>      7 days ago     /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.19     0B
  <missing>      7 days ago     /bin/sh -c addgroup -g 1000 node     && addu…   135MB
  <missing>      7 days ago     /bin/sh -c #(nop)  ENV NODE_VERSION=22.1.0      0B
  <missing>      3 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
  <missing>      3 months ago   /bin/sh -c #(nop) ADD file:37a76ec18f9887751…   7.38MB
  ```
  The rest is our build.

* Other commands
  `docker image history --no-trunc [image]` - Shows all the commands in details
  `docker image history --quiet [image]`
  `docker image history --no-trunc --quiet [image]`

# Saving and loading Images
* Save one or more images to a tar file:
  ```sh
  docker image save <IMAGE> > <FILE>.tar
  docker image save <IMAGE> -o <FILE>.tar
  docker image save <IMAGE> --output <FILE>.tar
  ```
* Load an image from a tar file:
  ```sh
  docker image load < <FILE>.tar
  docker image load -i <FILE>.tar
  docker image load --input <FILE>.tar
  ```
* Setup:
  ```sh
  mkdir output
  cd output
  ```
* Archive the ismgonza/weather-app:latest image:
  `docker image save ismgonza/weather-app:latest --output weather-app-latest.tar`

* Inspect the tar file:
  `tar tvf weather-app-latest.tar`

* Compress the tar file:
  `gzip weather-app-latest.tar`

* Delete the image:
  `docker image rm [USERNAME]/weather-app:latest`

* Load the weather-app image from a tar file:
  ```sh
  docker image load --input weather-app-latest.tar.gz
  docker image ls | grep [USERNAME]/weather-app
  docker image rm ismgonza/weather-app:latest
  docker image ls | grep ismgonza/weather-app
  ```
