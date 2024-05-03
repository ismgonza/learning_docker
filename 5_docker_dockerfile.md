## DockerFile

**Refs:**
* [Overview of best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### Dockerfile Example
```dockerfile
FROM ubuntu:22.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

In the example above, each instruction creates one layer:
* `FROM` creates a layer from the `ubuntu:22.04` Docker (base) image
* `COPY` adds files from your Docker client's current directory
* `RUN` builds your application with `make`
* `CMD` specifies what command to run within the container when it starts

### Notes
* Keep containers as ephemeral as possible
* Avoid unnecessary files
* Keep container as small as possible
* Use .dockerignore to ignore files and directories so they dont get copied over
* Use multi-stage builds
* Decouple applications
* Minimize the number of layers
* Sort the layers alphanumerically
* Use cache images

### Docker Commands
* Check -> [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
* `WORKDIR`: sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instruction that follow in the Dockerfile
* `EXPOSE`: ports can be separated by spaces
### Creating a DockerFile using instructions

* Create a working directory to save all important files
```sh
mkdir weather-app
cd weather-app
vim Dockerfile
```
* Create the dockerfile
```dockerfile
# Create Image from weather-app
    FROM node
    LABEL org.company.version=v1.1
    RUN mkdir -p /var/node
    ADD src/ /var/node/        # copies files from src/ to /var/node/
    WORKDIR /var/node
    RUN npm install
    EXPOSE 3000
    CMD ./bin/www
```
* Build the image, use -t to specify the name and tag of the image
`docker image build -t linuzacademy/weather-app:v1 .`

### Using ENVironments Variables
* Lets say we want to create two similar containers but one for DEV and one for PROD
```dockerfile
    FROM node
    LABEL org.company.version=v1.1
    ENV NODE_ENV="development"
    ENV PORT 3000
    RUN mkdir -p /var/node
    ADD src/ /var/node/
    WORKDIR /var/node
    RUN npm install
    EXPOSE $PORT
    CMD ./bin/www
```
* Build the image
  `docker image build -t linuxacademy/weather-app:v2 .`
* If i want to change any of the env variables when running the container i can do:
```sh
    docker container run -d \
    --name weather-app2 \
    -p 8082:3001 \
    --env PORT=3001 \
    --env NODE_ENV=production \
    linuxacademy/weather-app:v2
```

### Using ARGuments
```dockerfile
    FROM node
    LABEL org.company.version=v1.1
    ARG SRC_DIR=/var/node
    RUN mkdir -p $SRC_DIR
    ADD src/ $SRC_DIR
    WORKDIR $SRC_DIR
    RUN npm install
    EXPOSE $PORT
    CMD ./bin/www
```
* Change arguments when building the image
```sh
    docker image build -t \
    linuxacademy/weather-app:v3 \
    --build -arg SRC_DIR=/var/code \
    .
```
* Create the docker container
```sh
    docker container run -d \
    --name weather-app3 \
    -p 8085:3000 \
   linuxacademy/weather-app:v3
```

# TODO
try with another image 