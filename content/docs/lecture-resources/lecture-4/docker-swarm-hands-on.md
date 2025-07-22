---
title: "Docker Swarm Hands On"
weight: 430
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# 🛠️ Docker Swarm: Hands-On Lab

This practical guide will walk you through setting up a Docker Swarm cluster with one manager node and two worker nodes, then deploying and managing services on it.

## 🎯 Lab Objectives

- Set up a three-node Docker Swarm cluster
- Initialize the Swarm on the manager node
- Join worker nodes to the Swarm
- Deploy and scale services
- Inspect and manage the cluster

## 🖥️ Lab Environment

For this lab, we'll use three machines (physical, virtual, or cloud instances):

- **Manager Node**: Controls the Swarm cluster
- **Worker Node 1**: Runs containerized workloads
- **Worker Node 2**: Runs containerized workloads

### Prerequisites

- Three Linux machines with network connectivity between them
- All machines must be in the same network (e.g., 10.0.0.0/16)
- The following ports must be open between all nodes:
  - TCP port 2377 - Cluster management communications
  - TCP and UDP port 7946 - Node-to-node communication
  - UDP port 4789 - Overlay network traffic

## 📋 Step 1: Install Docker on All Nodes

Run the following commands on all three nodes:

```bash
# Remove any existing Docker installations
sudo apt remove docker docker.io containerd runc

# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y ca-certificates curl gnupg

# Set up Docker's apt repository
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin

# Add your user to the docker group to run Docker without sudo
sudo groupadd docker
sudo usermod -aG docker $USER

# Apply group changes to current shell session
newgrp docker

# Verify Docker installation
docker --version
docker run hello-world

# IP forwarding for routing mesh
sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1

```

## 🚀 Step 2: Initialize the Swarm on Manager Node

On the manager node only:

```bash
# Initialize the swarm with the manager node's IP address
docker swarm init --advertise-addr <MANAGER-IP>

# The output will include a token for worker nodes to join
# Save this token for the next step
```

The output will look like:

```
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

## 🔄 Step 3: Join Worker Nodes to the Swarm

On each worker node, run the join command from the previous step's output:

```bash
# Example (use the actual token from your manager node)
docker swarm join --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c <MANAGER-IP>:2377
```

The output should confirm:

```
This node joined a swarm as a worker.
```

## 🔍 Step 4: Verify the Swarm Cluster

On the manager node:

```bash
# List all nodes in the swarm
docker node ls
```

You should see output similar to:

```
ID                            HOSTNAME            STATUS    AVAILABILITY    MANAGER STATUS    ENGINE VERSION
dxn1zf6l61qsb1josjja83ngz *  manager             Ready     Active          Leader            20.10.17
yu3hbegvwsdpy9esh4f9fmq7z    worker1             Ready     Active                            20.10.17
ca7a0bxe9gc2hm5kixwz2tgpy    worker2             Ready     Active                            20.10.17
```

## 🌐 Step 5: Deploy a Service to the Swarm

On the manager node:

```bash
# Create a service with 3 replicas
docker service create --name web-server \
  --replicas 3 \
  --publish 8080:80 \
  nginx:latest
```

## 📊 Step 6: Manage and Inspect the Service

On the manager node:

```bash
# List all services
docker service ls

# View detailed information about the service
docker service ps web-server

# Scale the service to 5 replicas
docker service scale web-server=5

# Check service logs
docker service logs web-server

# Update the service to a newer image version
docker service update --image nginx:1.21 web-server
```

## 🧪 Step 7: Test the Service

Open a web browser and navigate to:
- `http://<MANAGER-IP>:8080`
- `http://<WORKER1-IP>:8080`
- `http://<WORKER2-IP>:8080`

You should see the Nginx welcome page on all three addresses due to the routing mesh.

## 🔄 Step 8: Create a Multi-Service Stack

Create a file named `docker-compose.yml` on the manager node:

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
    ports:
      - "8080:80"
    networks:
      - webnet

  visualizer:
    image: dockersamples/visualizer:latest
    ports:
      - "8081:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet

networks:
  webnet:
```

Deploy the stack:

```bash
# Deploy the stack
docker stack deploy -c docker-compose.yml myapp

# List all stacks
docker stack ls

# List services in the stack
docker stack services myapp

# List tasks in the stack
docker stack ps myapp
```

## 🧹 Step 9: Clean Up (When Finished)

```bash
# Remove the stack
docker stack rm myapp

# Remove the service
docker service rm web-server

# Leave the swarm (on worker nodes)
docker swarm leave

# Force leave and remove the swarm (on manager node)
docker swarm leave --force
```

## 🎓 Key Takeaways

- Docker Swarm provides native clustering and orchestration for Docker
- The Swarm consists of manager nodes (control plane) and worker nodes (data plane)
- Services are the primary unit of deployment in Swarm
- Swarm's routing mesh ensures services are accessible from any node in the cluster
- Docker Compose files can be used to deploy multi-service applications as stacks
