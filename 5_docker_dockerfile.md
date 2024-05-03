## DockerFile

**Refs:**
* [Overview of best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
* [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

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
* 