---
title: "Docker Best Practices"
weight: 340
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# Docker Best Practices

This document outlines recommended practices for effectively developing, deploying, and managing Docker containers in production environments.

## Image Building Best Practices

### Use Specific Base Image Tags

Always use specific version tags for base images rather than `latest` to ensure consistency and reproducibility.

```dockerfile
# Bad practice
FROM node:latest

# Good practice
FROM node:18.16.0-slim
```

### Keep Images Small

- Use lightweight base images (Alpine, slim variants)
- Minimize layers by combining related commands
- Remove unnecessary files in the same layer they were created

```dockerfile
# Bad practice
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN pip install requests

# Good practice
FROM python:3.11-alpine
RUN pip install --no-cache-dir requests
```

### Use Multi-Stage Builds

Use multi-stage builds to separate build-time dependencies from runtime dependencies, resulting in smaller final images.

```dockerfile
# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["npm", "start"]
```

### Leverage Build Cache Effectively

Order Dockerfile instructions from least to most frequently changing to optimize build cache usage.

```dockerfile
# Optimal order for caching
FROM node:18-alpine
WORKDIR /app

# Rarely changes
COPY package*.json ./
RUN npm ci

# More frequently changes
COPY . .
RUN npm run build

CMD ["npm", "start"]
```

### Avoid Running as Root

Run containers with a non-root user for security.

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY --chown=node:node . .
USER node
CMD ["node", "server.js"]
```

## Container Runtime Best Practices

### Use Environment Variables for Configuration

Externalize configuration using environment variables.

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]

# Run with: docker run -e DB_HOST=mongodb -e API_KEY=12345 my-app
```

### Set Health Checks

Define health checks to enable automated monitoring and container replacement.

```dockerfile
FROM nginx:alpine
COPY ./config/nginx.conf /etc/nginx/conf.d/default.conf
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

### Implement Proper Logging

Configure applications to log to stdout/stderr for integration with Docker's logging system.

```dockerfile
# Application should log to stdout/stderr
# Then view logs with: docker logs container_name
```

### Define Resource Limits

Always set resource constraints for containers in production.

```bash
# Set memory and CPU limits
docker run --memory=512m --cpus=0.5 my-application
```

### Use Volumes for Persistent Data

Use named volumes for data that needs to persist beyond container lifecycle.

```bash
# Create a named volume
docker volume create my-data

# Use the volume
docker run -v my-data:/app/data my-application
```

## Security Best Practices

### Scan Images for Vulnerabilities

Regularly scan your Docker images for known vulnerabilities.

```bash
# Example using Trivy
trivy image myapp:1.0
```

### Sign and Verify Images

Sign your images and verify signatures before deployment for supply chain security.

```bash
# Sign image
docker trust sign myapp:1.0

# Only run verified images
docker trust inspect --pretty myapp:1.0
```

### Keep Base Images Updated

Regularly update base images to include security patches.

```bash
# Pull latest version of the specific tag you use
docker pull node:18-alpine
```

### Limit Capabilities

Run containers with minimal required Linux capabilities.

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE my-app
```

### Use Read-Only Filesystem

Mount the container filesystem as read-only when possible.

```bash
docker run --read-only my-application
```

## Development Workflow Best Practices

### Use Docker for Local Development

Create development-specific Dockerfiles for your application.

```dockerfile
# Dockerfile.dev
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

Run with:
```bash
docker build -f Dockerfile.dev -t myapp-dev .
docker run -v $(pwd)/src:/app/src -p 3000:3000 myapp-dev
```

### Implement CI/CD for Docker Images

Automate building, testing, and deploying Docker images.

```yaml
# Example GitHub Actions workflow
name: Docker Build and Push

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build and test
        run: |
          docker build -t myapp:test .
          docker run myapp:test npm test
      
      - name: Push to registry
        if: success()
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker tag myapp:test myorg/myapp:${{ github.sha }}
          docker push myorg/myapp:${{ github.sha }}
```

### Use Docker BuildKit

Enable BuildKit for faster, more efficient builds.

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1
docker build .
```

## Monitoring and Maintenance

### Set Up Container Monitoring

Implement monitoring for all Docker containers in production.

```bash
# Example using Prometheus and cAdvisor
docker run -d \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:latest
```

### Log Aggregation

Implement centralized logging for all containers.

```bash
# Run ELK stack for log aggregation
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  elasticsearch:7.14.0

docker run -d \
  --name logstash \
  --link elasticsearch \
  -v "$PWD/logstash.conf:/usr/share/logstash/pipeline/logstash.conf" \
  logstash:7.14.0

docker run -d \
  --name kibana \
  --link elasticsearch \
  -p 5601:5601 \
  kibana:7.14.0
```

### Regular Cleanup

Implement regular cleanup of unused Docker objects.

```bash
# Create a cleanup cron job
0 0 * * * docker system prune -af --volumes > /var/log/docker-cleanup.log 2>&1
```

## Production Deployment Considerations

### Use Orchestration Tools

For production environments with multiple containers, use orchestration tools like Kubernetes or Docker Swarm.

### Implement Rolling Updates

Configure services for zero-downtime updates.

```bash
# Docker Swarm example
docker service create --name myapp --replicas 3 myapp:1.0
docker service update --image myapp:1.1 myapp
```

### Backup Strategies

Implement regular backups for all persistent data.

```bash
# Example backup script for a volume
docker run --rm -v my_volume:/data -v $(pwd)/backup:/backup alpine \
  tar -czf /backup/my_volume_$(date +%Y%m%d_%H%M%S).tar.gz -C /data .
```

## Conclusion

Following these best practices will help you build more secure, efficient, and maintainable Docker-based applications. Remember that Docker is constantly evolving, so it's important to stay updated with the latest recommendations and security advisories. 