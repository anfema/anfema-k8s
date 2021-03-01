# Creating a local docker registry

## Running the registry

This is simple, just run the registry container in your docker daemon:

```bash
docker run -d -p 5000:5000 --restart=always --name registry -v /much/space:/var/lib/registry registry:2
```

Replace `/much/space` with a path to a fast storage medium with plenty of space
to save the docker images of the registry to.

Leave out the `--restart=always` if you do not want to run the registry on
system startup.

## Pushing images to the registry

Just run the following command in the same directory as your `Dockerfile`:

```bash
docker push localhost:5000/<name>
```
## Copying an image from another registry

If you want to copy an image from another registry into your private registry
you can absolutely do so.

```bash
docker pull <image>
docker tag <image> localhost:5000/<image>
docker push localhost:5000/<image>
```

Removal for local images is as easy as well:

```bash
docker image remove localhost:5000/<image>
```

## Running a registry with UI

You can use the docker-compose script in `/scripts/Docker-Registry.yaml` to
automatically deploy a registry with running UI on `http://localhost:8080`.

## Garbage collect images from registry

If you're running the docker registry script from above you can call the
garbage collector like this:

```bash
docker exec scripts_registry_1 registry garbage-collect /etc/docker/registry/config.yml
```