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

Removal is as easy:

```bash
docker image remove localhost:5000/<image>
```