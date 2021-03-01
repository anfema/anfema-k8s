# Creating docker images

To create a docker image you define a base-image to base your work on and then
you tell docker what commands to run on that base-image to create your overlay.

If you pack the resulting container into an image, only the overlay data is
persisted, the base image remains separate and so it can be cached seperately.

## Dockerfile

At first the example, description follows

```Dockerfile
# syntax=docker/dockerfile
FROM python:3.8
LABEL maintainer="j.schriewer@anfe.ma"

ENV PYTHONUNBUFFERED 1
ENV DJANGO_ENV dev

WORKDIR /code/

ADD Pipfile /code/
ADD Pipfile.lock /code/
RUN pip install --upgrade pip pipenv

RUN pipenv sync
COPY django_project /code/django_project
COPY django_app /code/django_app

VOLUME /code/media
EXPOSE 8000/tcp
USER django:django
ENTRYPOINT ["/usr/bin/python", "manage.py"]
CMD ["runserver", "0.0.0.0:8000"]
```

- `FROM` the base image
- `LABEL` customized labels, usually `maintainer` is supplied
- `ENV` sets an environment variable, can be referenced with brace style in
  other commands in the dockerfile (e.g. `${DJANGO_ENV}`)
- `WORKDIR` creates a directory and changes to it
- `ADD` add a single file to the image
- `RUN` runs commands
- `COPY` copies complete directory structures into the container
- `VOLUME` makes a directory available to mount an external persistent volume to
- `EXPOSE` exposes a port
- `USER` creates and selects a user to run the container command under
- `ENTRYPOINT` the command to run with `docker run --name <container> <args>`
- `CMD` the command which is run when running `docker start <container>`. If
  given in exec form (as an array) it will be appended to the `ENTRYPOINT`

With this container definition you will be able to run commands on `manage.py`
by just using a simple `docker run` command as well as just start the container
and have it running the service.

**Attention**: To run a command in a running container user `docker run`, if you
use `docker exec` docker will create a new container, execute the command and
then tear the container down afterwards!

## Building and tagging for registry

To build a docker image and tag it to be able to be pushed to a registry, use
the following command:

```bash
docker build -t localhost:5000/<image>:<version> -t localhost:5000/<image>:latest .
```

This will tag the built image to be pushed to the `localhost:5000` registry
with a version and the `latest` tag as a shortcut. Building an image needs a
running docker environment, as not the cli is building the image but the daemon
is.

**Attention**: If you have Docker > 18.09 you can use BuildKit to accelerate and
optimize image file generation. Export the `DOCKER_BUILDKIT=1` environment
variable to use it.

## .dockerignore

To speed up builds you can add a `.dockerignore` file to skip any directory
while copying to the container. It works just like `.gitignore`.

## Manage images on your device

If you often build images on your device you will at some time feel the need of
cleaning up all the cruft that accumulated. To do so:

List all images on your device:

```bash
docker images
```

Determine which you want to delete and then run:

```bash
docker rmi <image-id>
```