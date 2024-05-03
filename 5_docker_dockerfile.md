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
vim dockerfile
```
* Create the dockerfile
```dockerfile
# Create Image from weather-app
FROM node
LABEL org.company.version=v1.1
RUN mkdir -p /var/node
ADD src/ /var/node/        # copies files from src/ to /var/node/
WORKDIR /var/node
RUN npn install
EXPOSE 3000
CMD ./bin/www
```
* Build the image, use -t to specify the name and tag of the image
`docker image build -t linuzacademy/weather-app:v1 .`

