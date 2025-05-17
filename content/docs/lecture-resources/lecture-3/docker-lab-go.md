---
title: "Docker Lab with Go"
weight: 325
bookFlatSection: false
bookToc: true
bookHidden: false
bookCollapseSection: false
bookComments: false
bookSearchExclude: false
---

# Docker Hands-On Lab with Go

This lab provides practical exercises using Go to reinforce the Docker concepts covered in the lecture.

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

Let's run an interactive container using the official Alpine image:

```bash
docker run -it --rm alpine sh
```

Inside the container, try some commands:
```bash
ls
cat /etc/os-release
apk update
```

Type `exit` to leave the container.

## Exercise 2: Building a Go Application with Docker

### 2.1 Create a Simple Go Web Server

Create a directory for your project:

```bash
mkdir docker-go-lab
cd docker-go-lab
```

Create a file named `main.go`:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"os"
)

type ServerInfo struct {
	Hostname  string `json:"hostname"`
	GoVersion string `json:"go_version"`
	Container bool   `json:"container"`
}

func main() {
	// Get hostname
	hostname, err := os.Hostname()
	if err != nil {
		hostname = "unknown"
	}

	// Define HTTP handlers
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "text/plain")
		fmt.Fprintf(w, "Hello from Docker!\n")
		fmt.Fprintf(w, "This is a Go application running in a container.\n")
		fmt.Fprintf(w, "Container hostname: %s\n", hostname)
	})

	http.HandleFunc("/api/info", func(w http.ResponseWriter, r *http.Request) {
		info := ServerInfo{
			Hostname:  hostname,
			GoVersion: "1.20", // Hardcoded for simplicity, in reality use runtime.Version()
			Container: true,
		}

		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(info)
	})

	// Start server
	port := 8080
	fmt.Printf("Starting server on port %d...\n", port)
	log.Fatal(http.ListenAndServe(fmt.Sprintf(":%d", port), nil))
}
```

### 2.2 Create a Go Module

Initialize a Go module:

```bash
go mod init docker-go-lab
```

### 2.3 Create a Dockerfile

Create a file named `Dockerfile`:

```dockerfile
# Build stage
FROM golang:1.20-alpine AS build

WORKDIR /app

# Copy go module files first for better caching
COPY go.mod ./
# Copy source code
COPY main.go ./

# Build the application
RUN go build -o server .

# Runtime stage
FROM alpine:latest

WORKDIR /app

# Copy the binary from the build stage
COPY --from=build /app/server /app/server

# Expose the port
EXPOSE 8080

# Run the application
CMD ["/app/server"]
```

### 2.4 Build and Run Your Image

Build your Docker image:

```bash
docker build -t go-web-app .
```

Run the container:

```bash
docker run -d -p 8080:8080 --name go-app go-web-app
```

Access your application:

```bash
# Using curl
curl http://localhost:8080

# Using curl for JSON API
curl http://localhost:8080/api/info
```

You can also visit `http://localhost:8080` in your browser.

### 2.5 Explore Your Container

Check the running containers:

```bash
docker ps
```

View container logs:

```bash
docker logs go-app
```

Execute commands in the running container:

```bash
docker exec -it go-app sh
```

Stop and remove your container:

```bash
docker stop go-app
docker rm go-app
```

## Exercise 3: Multi-Stage Builds and Optimization

### 3.1 Create an Optimized Dockerfile

Let's create an even more optimized Dockerfile that produces a smaller image:

```dockerfile
# Build stage
FROM golang:1.20-alpine AS build

WORKDIR /app

# Copy go module files
COPY go.mod ./
COPY main.go ./

# Build with optimizations
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o server .

# Runtime stage - using scratch (empty) image
FROM scratch

# Copy SSL certificates for HTTPS requests
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

WORKDIR /app

# Copy the binary from the build stage
COPY --from=build /app/server /app/server

# Expose the port
EXPOSE 8080

# Run the application
CMD ["/app/server"]
```

Build and tag this optimized image:

```bash
docker build -t go-web-app:optimized -f Dockerfile.optimized .
```

Compare the image sizes:

```bash
docker images | grep go-web-app
```

## Exercise 4: Docker Networks and Volumes with Go

### 4.1 Create a Stateful Go Application

Create a new file named `stateful.go`:

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"path/filepath"
	"sync"
)

// Counter represents a simple counter with mutex for concurrent access
type Counter struct {
	Value int    `json:"value"`
	Path  string `json:"-"`
	mu    sync.Mutex
}

// NewCounter creates a new counter and loads its value from file if available
func NewCounter(dataPath string) *Counter {
	c := &Counter{
		Value: 0,
		Path:  dataPath,
	}

	// Create directory if it doesn't exist
	dir := filepath.Dir(dataPath)
	if err := os.MkdirAll(dir, 0755); err != nil {
		log.Printf("Error creating directory: %v", err)
	}

	// Try to load existing counter value
	if data, err := ioutil.ReadFile(dataPath); err == nil {
		var storedCounter Counter
		if err := json.Unmarshal(data, &storedCounter); err == nil {
			c.Value = storedCounter.Value
		}
	}

	return c
}

// Increment increases the counter and saves it
func (c *Counter) Increment() int {
	c.mu.Lock()
	defer c.mu.Unlock()
	
	c.Value++
	
	// Save to file
	data, err := json.Marshal(c)
	if err != nil {
		log.Printf("Error marshaling counter: %v", err)
		return c.Value
	}
	
	if err := ioutil.WriteFile(c.Path, data, 0644); err != nil {
		log.Printf("Error writing counter: %v", err)
	}
	
	return c.Value
}

// GetValue returns the current value
func (c *Counter) GetValue() int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.Value
}

func main() {
	// Create a counter that persists to a file
	counter := NewCounter("/data/counter.json")
	
	// Get hostname for display
	hostname, _ := os.Hostname()
	
	// Define HTTP handlers
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "text/plain")
		fmt.Fprintf(w, "Hello from Docker!\n")
		fmt.Fprintf(w, "This is a stateful Go application with persistence.\n")
		fmt.Fprintf(w, "Container hostname: %s\n", hostname)
		fmt.Fprintf(w, "Current count: %d\n", counter.GetValue())
		fmt.Fprintf(w, "Visit /increment to increase the counter.\n")
	})
	
	http.HandleFunc("/increment", func(w http.ResponseWriter, r *http.Request) {
		newValue := counter.Increment()
		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(map[string]interface{}{
			"value": newValue,
			"hostname": hostname,
		})
	})
	
	// Start server
	port := 8080
	fmt.Printf("Starting server on port %d with data persistence...\n", port)
	log.Fatal(http.ListenAndServe(fmt.Sprintf(":%d", port), nil))
}
```

Create a Dockerfile for the stateful application:

```dockerfile
FROM golang:1.20-alpine AS build

WORKDIR /app

COPY stateful.go .

RUN go build -o stateful-app stateful.go

FROM alpine:latest

WORKDIR /app

COPY --from=build /app/stateful-app .

# Create a volume mount point
VOLUME /data

EXPOSE 8080

CMD ["/app/stateful-app"]
```

### 4.2 Run with Persistent Volume

Build the stateful application:

```bash
docker build -t go-stateful-app -f Dockerfile.stateful .
```

Create a volume:

```bash
docker volume create go-app-data
```

Run the container with the volume:

```bash
docker run -d -p 8080:8080 -v go-app-data:/data --name stateful-app go-stateful-app
```

Test the persistence:
1. Visit `http://localhost:8080/increment` several times
2. Stop and remove the container
3. Start a new container with the same volume
4. Verify the counter value persisted

```bash
# Stop and remove the container
docker stop stateful-app
docker rm stateful-app

# Start a new container with the same volume
docker run -d -p 8080:8080 -v go-app-data:/data --name stateful-app-2 go-stateful-app

# Check if the counter value persisted
curl http://localhost:8080
```

## Conclusion

This lab has covered:
- Running containers with Docker
- Building Go applications for containerized environments
- Using multi-stage builds for optimization
- Working with Docker volumes for persistence

These practices are essential for developing modern, containerized Go applications.

## Challenge Exercise

Build a multi-container application with these components:
1. A Go API server that handles requests
2. A separate data storage container
3. Connect them using Docker networks for communication
4. Implement proper volume mounts for data persistence

Both containers should be configured to restart automatically if they crash or if Docker restarts. 