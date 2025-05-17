---
title: "Docker Hands-On Lab"
weight: 320
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# Docker Hands-On Lab

This lab provides practical exercises to reinforce the Docker concepts covered in the lecture. These hands-on activities will help you build confidence in using Docker for development and deployment scenarios.

## Prerequisites

- Docker Engine installed on your system
- Basic command line familiarity
- Text editor of your choice

## Exercise 1: Getting Started with Docker

### 1.1 Verify Docker Installation

Ensure Docker is properly installed on your system:

```bash
docker --version
docker info
```

### 1.2 Running Your First Container

Let's run an interactive container using the official Ubuntu image:

```bash
docker run -it --rm ubuntu bash
```

Inside the container, try some commands:
```bash
ls
cat /etc/os-release
apt update
```

Type `exit` to leave the container.

## Exercise 2: Building Custom Images

### 2.1 Create a Simple Python Web Application

Create a directory for your project:

```bash
mkdir docker-lab
cd docker-lab
```

Create a file named `app.py`:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import socket
import json

class WebServer(BaseHTTPRequestHandler):
    def _set_response(self, content_type="text/plain"):
        self.send_response(200)
        self.send_header('Content-type', content_type)
        self.end_headers()
        
    def do_GET(self):
        if self.path == '/':
            # Serve the main page as plain text
            self._set_response("text/plain")
            hostname = socket.gethostname()
            response = f"""
Hello from Docker!
This page is being served from a Docker container.
Container hostname: {hostname}
            """
            self.wfile.write(response.encode('utf-8'))
        elif self.path == '/api/info':
            # Serve JSON API response
            self._set_response("application/json")
            data = {
                "hostname": socket.gethostname(),
                "python_version": socket.python_version(),
                "container": True
            }
            self.wfile.write(json.dumps(data).encode('utf-8'))
        else:
            self._set_response()
            self.wfile.write(b"404 - Not Found")

def run(server_class=HTTPServer, handler_class=WebServer, port=3000):
    server_address = ('', port)
    httpd = server_class(server_address, handler_class)
    print(f"Starting server on port {port}...")
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        pass
    httpd.server_close()
    print("Server stopped.")

if __name__ == '__main__':
    run()
```

### 2.2 Create a Dockerfile

Create a file named `Dockerfile`:

```dockerfile
# Use Python as the base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy application file
COPY app.py .

# Expose the port the app runs on
EXPOSE 3000

# Command to run the application
CMD ["python", "app.py"]
```

### 2.3 Build and Run Your Image

Build your Docker image:

```bash
docker build -t my-python-app .
```

Run the container:

```bash
docker run -d -p 3000:3000 --name web-app my-python-app
```

Test your application:

```bash
# Using curl for plain text response
curl http://localhost:3000

# Using curl for JSON API
curl http://localhost:3000/api/info
```

### 2.4 Explore Your Container

Check the running containers:

```bash
docker ps
```

View container logs:

```bash
docker logs web-app
```

Execute commands in the running container:

```bash
docker exec -it web-app bash
```

Stop and remove your container:

```bash
docker stop web-app
docker rm web-app
```

## Exercise 3: Advanced Docker Concepts

### 3.1 Docker Networks

Create a custom bridge network:

```bash
docker network create mynetwork
```

Run containers on the network:

```bash
docker run -d --name container1 --network mynetwork alpine sleep infinity
docker run -d --name container2 --network mynetwork alpine sleep infinity
```

Test container communication:

```bash
docker exec container1 ping -c 3 container2
```

List networks and inspect:

```bash
docker network ls
docker network inspect mynetwork
```

Clean up:

```bash
docker stop container1 container2
docker rm container1 container2
docker network rm mynetwork
```

### 3.2 Docker Volumes

Create a named volume:

```bash
docker volume create mydata
```

Run a container with the volume:

```bash
docker run -d --name vol-test -v mydata:/data alpine sh -c "while true; do date >> /data/dates.txt; sleep 10; done"
```

Inspect the volume data:

```bash
docker exec vol-test cat /data/dates.txt
```

Use a different container to access the same volume:

```bash
docker run --rm -v mydata:/data alpine cat /data/dates.txt
```

Clean up:

```bash
docker stop vol-test
docker rm vol-test
docker volume rm mydata
```

## Conclusion

This lab has covered:
- Running containers with Docker
- Building custom Docker images
- Using Docker networks and volumes

Continue experimenting with Docker and explore other advanced topics like multi-stage builds, container orchestration with Kubernetes, and CI/CD integration.

## Challenge Exercise 

### Python API with Persistent Data

Create a multi-container application with these components:
1. A Python Flask API server that stores data in a persistent volume
2. Connect containers using a custom Docker network
3. Configure the containers to start automatically when Docker starts

Here's a starter for your Flask API (`app.py`):

```python
from flask import Flask, request, jsonify
import os
import json
import socket

app = Flask(__name__)
DATA_FILE = '/data/items.json'

# Create data directory if it doesn't exist
os.makedirs(os.path.dirname(DATA_FILE), exist_ok=True)

# Initialize with empty data if file doesn't exist
if not os.path.exists(DATA_FILE):
    with open(DATA_FILE, 'w') as f:
        json.dump([], f)

@app.route('/items', methods=['GET'])
def get_items():
    with open(DATA_FILE, 'r') as f:
        return jsonify(json.load(f))

@app.route('/items', methods=['POST'])
def add_item():
    new_item = request.json
    with open(DATA_FILE, 'r') as f:
        items = json.load(f)
    
    items.append(new_item)
    
    with open(DATA_FILE, 'w') as f:
        json.dump(items, f)
    
    return jsonify({"status": "success", "message": "Item added"})

@app.route('/info', methods=['GET'])
def get_info():
    return jsonify({
        "hostname": socket.gethostname(),
        "items_count": len(json.load(open(DATA_FILE, 'r'))),
        "container": True
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
``` 