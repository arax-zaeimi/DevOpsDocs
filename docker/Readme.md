# **Docker**

## **Pull and Start**

```
docker pull <IMAGE_NAME>:<TAG>
```

```
docker run -p <SRC_PORT>:<TARGET_PORT> -e <ENVIRONMENTAL_VARIABLE[Key Value Pair]> <IMAGE_NAME>:<TAG> --name <CONTAINER_NAME> --net <NETWORK_NAME>
```

Run container in detached mode:

```
docker run -d <IMAGE_NAME>
```

```
docker stop <CONTAINER_ID>
```

```
docker start <CONTAINER_ID>
```

List of all containers:

```
docker ps -a
```

Images list on local:

```
docker images
```

## **Troubleshooting Container**

```
docker logs <CONTAINER_ID>

docker logs <CONTAINER_NAME>
```

Show only the last logs:

```
docker logs <CONTAINER_ID> | tail
```

Stream the logs:

```
docker logs <CONTAINER_ID> -f
```

```
docker exec -it <CONTAINER_ID> /bin/bash
```

## **Docker Network**

```
docker network ls
```

```
docker network create <NAME>
```

## **Docker Compose**

When you want to run a docker image, you need a bunch of commands to set the attributes of the container. Using docker compose you can have these settings and attributes in a structured YAML file format.

```
version: '3'
services:
  CONTAINER_NAME:
    image: IMAGE_NAME
    ports:
      - 8080:8080 # HOST:CONTAINER
    environment:
      - name=value
    volumes:
      - VOLUME_NAME:CONTAINER_PATH:MODE
volumes:
  VOLUME_NAME
```

```
docker-compose -f <DOCKER_COMPOSE_FILE> up
```

```
docker-compose -f <DOCKER_COMPOSE_FILE> down
```

## **Packaging**

Dockerfile a blueprint for building images.

```
docker bulid -t <IMAGE_NAME>:<TAG> -f <DOCKERFILE_PATH>
```

```
docker rmi <IMAGE_ID>
```

## **Docker Volume**

There are 3 methods to mount volumes:

- Specify host and container directories
- Anonymouse Volumes
- Named Volumes

1. In this method, you specifically mention a physical path on the host to be mounted:

```
-v <HOST_DIRECTORY>:<CONTAINER_DIRECTORY>
```

2. Anonymouse Volumes: In this method you just specify the directory on the container, and the docker mounts a volume with a random name to the container

```
-v <CONTAINER_DIRECTORY>
```

3. Named Volumes: It mounts random volume to the container, but you define a specific name for this volume

```
-v <NAME>:<CONTAINER_DIRECTORY>
```

**Note:** The 3rd method is the most popular method in production.
