---
title: "Docker Command Cheat Sheet"
weight: 330
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# Docker Command Cheat Sheet

This cheat sheet provides a quick reference for commonly used Docker commands.

## Docker System Commands

| Command | Description |
|---------|-------------|
| `docker version` | Display Docker version information |
| `docker info` | Display system-wide information |
| `docker system prune` | Remove all unused containers, networks, images, and build cache |
| `docker system df` | Show Docker disk usage |

## Docker Image Commands

| Command | Description |
|---------|-------------|
| `docker images` | List all images |
| `docker pull <image-name>` | Pull an image from registry |
| `docker build -t <name:tag> <path>` | Build an image from Dockerfile |
| `docker rmi <image-id>` | Remove an image |
| `docker image prune` | Remove unused images |
| `docker tag <source-image> <target-image>` | Create a tag for image |
| `docker history <image-name>` | Show history of an image |
| `docker save -o <file.tar> <image-name>` | Save image to a tar archive |
| `docker load -i <file.tar>` | Load image from a tar archive |
| `docker inspect <image-id>` | Display detailed information about an image |

## Docker Container Commands

| Command | Description |
|---------|-------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker run <image-name>` | Run a container |
| `docker run -it <image-name> <command>` | Run container interactively |
| `docker run -d <image-name>` | Run container in detached mode |
| `docker run -p <host-port>:<container-port> <image-name>` | Run container with port mapping |
| `docker run -v <host-path>:<container-path> <image-name>` | Run container with volume mount |
| `docker run --name <container-name> <image-name>` | Run container with a specific name |
| `docker run --rm <image-name>` | Run container and remove it when it exits |
| `docker stop <container-id>` | Stop a running container |
| `docker start <container-id>` | Start a stopped container |
| `docker restart <container-id>` | Restart a container |
| `docker rm <container-id>` | Remove a container |
| `docker rm -f <container-id>` | Force remove a running container |
| `docker exec -it <container-id> <command>` | Execute a command in a running container |
| `docker logs <container-id>` | Fetch the logs of a container |
| `docker logs -f <container-id>` | Follow log output of a container |
| `docker cp <container-id>:<container-path> <host-path>` | Copy files from container to host |
| `docker cp <host-path> <container-id>:<container-path>` | Copy files from host to container |
| `docker stats` | Display live container resource usage |
| `docker inspect <container-id>` | Display detailed information about a container |
| `docker top <container-id>` | Display the running processes of a container |
| `docker update --memory <limit> <container-id>` | Update container resources |

## Docker Network Commands

| Command | Description |
|---------|-------------|
| `docker network ls` | List all networks |
| `docker network create <network-name>` | Create a network |
| `docker network rm <network-name>` | Remove a network |
| `docker network inspect <network-name>` | Display detailed information about a network |
| `docker network connect <network-name> <container-id>` | Connect a container to a network |
| `docker network disconnect <network-name> <container-id>` | Disconnect a container from a network |
| `docker network prune` | Remove all unused networks |

## Docker Volume Commands

| Command | Description |
|---------|-------------|
| `docker volume ls` | List all volumes |
| `docker volume create <volume-name>` | Create a volume |
| `docker volume rm <volume-name>` | Remove a volume |
| `docker volume inspect <volume-name>` | Display detailed information about a volume |
| `docker volume prune` | Remove all unused volumes |

## Dockerfile Instructions

| Instruction | Description |
|-------------|-------------|
| `FROM` | Set base image |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files from host to container |
| `ADD` | Copy files from host or URL to container |
| `RUN` | Execute command during build |
| `ENV` | Set environment variable |
| `ARG` | Define build-time variable |
| `EXPOSE` | Expose port |
| `VOLUME` | Create mount point |
| `CMD` | Default command for container |
| `ENTRYPOINT` | Configure container executable |
| `LABEL` | Add metadata to image |
| `USER` | Set user for container |
| `HEALTHCHECK` | Check container health |

## Common Docker Run Options

| Option | Description |
|--------|-------------|
| `-d, --detach` | Run container in background |
| `-e, --env` | Set environment variables |
| `-p, --publish` | Publish container's port to the host |
| `-v, --volume` | Bind mount a volume |
| `--name` | Assign a name to the container |
| `--network` | Connect to a network |
| `--restart` | Restart policy (no, on-failure, always, unless-stopped) |
| `--rm` | Remove container when it exits |
| `-it` | Interactive mode with TTY |
| `--memory` | Memory limit |
| `--cpus` | Number of CPUs |

## Docker Command Examples

### Running a Nginx Web Server

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

### Building a Custom Image

```bash
docker build -t my-app:1.0 .
```

### Running a Container with Environment Variables

```bash
docker run -d -p 3000:3000 -e NODE_ENV=production --name my-node-app my-node-app
```

### Creating a Docker Network and Connecting Containers

```bash
docker network create my-network
docker run -d --name db --network my-network mongo
docker run -d --name app --network my-network -p 8000:8000 my-app
```

### Volume Mounting for Persistence

```bash
docker run -d -p 27017:27017 -v mongodb-data:/data/db --name mongodb mongo
```

### Running a MySQL Container with Environment Variables

```bash
docker run -d -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=secretpassword \
  -e MYSQL_DATABASE=mydb \
  -e MYSQL_USER=user \
  -e MYSQL_PASSWORD=password \
  -v mysql-data:/var/lib/mysql \
  --name mysql \
  mysql:8.0
``` 