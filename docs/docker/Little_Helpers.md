# Little helpers for docker

## Delete an image

```bash
docker images
```

and take the image ID, then

```bash
docker rmi <id>
```

if you get an error like `image is being used by stopped container`, delete all
stopped containers

```bash
docker container prune
```

or just one:

```bash
docker container ls -a
docker rm <container-id>
```

## Docker compose

Run a group of images:

```bash
docker-compose -f <compose file> up -d
```

Delete a group of images

```bash
docker-compose -f <compose file> down
```