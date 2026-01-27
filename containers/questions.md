# Container Interview Questions

**Q1: What is a Container?**

A container is a lightweight, standalone, executable package that includes everything needed to run a piece of software: code, runtime, system tools, libraries, and settings. Containers isolate software from its environment and ensure it works uniformly across different computing environments.

**Q2: How do containers differ from Virtual Machines?**

Containers share the host OS kernel and isolate the application processes, making them lightweight and fast to start. VMs include a full OS copy for each instance, requiring more resources and time to boot.

| Feature            | Containers        | Virtual Machines |
| ------------------ | ----------------- | ---------------- |
| **OS**             | Share host kernel | Full OS per VM   |
| **Size**           | MBs               | GBs              |
| **Startup**        | Seconds           | Minutes          |
| **Resource Usage** | Lightweight       | Heavy            |
| **Isolation**      | Process-level     | Hardware-level   |

**Q3: What is Docker?**

Docker is a platform for developing, shipping, and running applications in containers. It provides tools and a runtime to build container images, manage containers, and orchestrate containerized applications.

**Q4: Explain the Docker architecture**

Docker follows a client-server architecture:

**Docker Client:**

- `docker` CLI tool that users interact with
- Sends commands to Docker daemon

**Docker Daemon (dockerd):**

- Runs on the host machine
- Manages Docker objects: images, containers, networks, volumes
- Listens for Docker API requests

**Docker Registry:**

- Stores Docker images (e.g., Docker Hub, private registries)
- `docker pull` downloads images from registry
- `docker push` uploads images to registry

**Q5: What is a Docker Image?**

A Docker image is a read-only template with instructions for creating a container. It contains the application code, runtime, libraries, dependencies, and configuration files. Images are built from a `Dockerfile` and can be stored in registries.

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Q6: What is the difference between an Image and a Container?**

- **Image**: A static, immutable template (blueprint)
- **Container**: A running instance of an image (the actual application)

Think of it like a class (image) vs an object (container) in programming.

**Q7: What is a Dockerfile?**

A Dockerfile is a text file containing a set of instructions to build a Docker image. Each instruction creates a layer in the image.

**Common Dockerfile Instructions:**

- `FROM`: Base image
- `WORKDIR`: Set working directory
- `COPY/ADD`: Copy files into the image
- `RUN`: Execute commands during build
- `EXPOSE`: Document which ports the container listens on
- `CMD`: Default command to run when container starts
- `ENTRYPOINT`: Configure container to run as an executable

**Q8: What is the difference between CMD and ENTRYPOINT?**

Both define what command runs when the container starts, but they work differently:

- **CMD**: Provides default arguments that can be overridden when running the container
- **ENTRYPOINT**: Sets the main command that always runs; arguments are appended to it

```dockerfile
# CMD - can be completely overridden
CMD ["echo", "Hello"]

# ENTRYPOINT - always runs, arguments are appended
ENTRYPOINT ["echo"]
CMD ["Hello"]  # Default argument
```

**Best Practice:** Use `ENTRYPOINT` for the main executable and `CMD` for default flags/arguments.

**Q9: What are Docker layers?**

Each instruction in a Dockerfile creates a read-only layer. When you run a container, Docker adds a writable layer on top. Layers are cached and reused, making builds faster and storage more efficient.

```dockerfile
FROM ubuntu:22.04      # Layer 1
RUN apt-get update     # Layer 2
RUN apt-get install -y python3  # Layer 3
COPY app.py /app/      # Layer 4
```

**Layer Caching:** If you change Layer 4, only Layer 4 needs to rebuild. Layers 1-3 are reused from cache.

**Q10: What is the difference between COPY and ADD?**

Both copy files from host to image, but `ADD` has extra features:

- **COPY**: Simple file/directory copy (preferred for clarity)
- **ADD**: Can extract tar archives and download URLs (avoid unless you need these features)

```dockerfile
COPY app.py /app/           # Recommended
ADD app.py /app/            # Works, but use COPY instead
ADD archive.tar.gz /data/   # Extracts archive (use case for ADD)
```

**Q11: What is Docker Compose?**

Docker Compose is a tool for defining and running multi-container applications. You use a `docker-compose.yml` file to configure your application's services, networks, and volumes.

```yaml
version: "3.8"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

**Common Commands:**

- `docker-compose up`: Start all services
- `docker-compose down`: Stop and remove containers
- `docker-compose ps`: List containers

**Q12: What are Docker volumes?**

Volumes are the preferred way to persist data generated and used by Docker containers. They exist outside the container's writable layer and persist even after the container is deleted.

**Types of Mounts:**

- **Volumes**: Managed by Docker, stored in `/var/lib/docker/volumes/`
- **Bind Mounts**: Mount a host directory/file into container
- **tmpfs**: Stored in memory only (non-persistent)

```bash
# Named volume
docker run -v my-volume:/app/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Create volume
docker volume create my-volume
```

**Q13: What is the difference between a Docker volume and a bind mount?**

- **Volume**: Managed by Docker, portable, works on all platforms
- **Bind Mount**: Direct mapping to host filesystem, tied to host directory structure

Use volumes for production; use bind mounts for development (live code reload).

**Q14: What are Docker networks?**

Docker networks enable communication between containers. By default, containers on the same network can communicate using container names as hostnames.

**Network Types:**

- **bridge** (default): Containers on same host communicate via private network
- **host**: Container uses host's network directly (no isolation)
- **none**: No networking
- **overlay**: Multi-host networking for Swarm/Kubernetes

```bash
# Create network
docker network create my-network

# Run containers on same network
docker run --network my-network --name web nginx
docker run --network my-network --name api node-app

# 'web' can reach 'api' using hostname 'api'
```

**Q15: How do you optimize Docker images?**

Best practices for smaller, faster images:

1. **Use minimal base images**: `alpine` instead of `ubuntu`
2. **Multi-stage builds**: Separate build and runtime environments
3. **Combine RUN commands**: Fewer layers
4. **Use .dockerignore**: Don't copy unnecessary files
5. **Order layers by change frequency**: Less frequently changed instructions first

```dockerfile
# Multi-stage build example
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/server.js"]
```

**Q16: What is a multi-stage build?**

Multi-stage builds use multiple `FROM` statements in a Dockerfile. Each stage can copy artifacts from previous stages, allowing you to separate build dependencies from runtime dependencies.

**Benefits:**

- Smaller final images (no build tools in production image)
- Better security (fewer attack surfaces)
- Cleaner Dockerfiles

**Q17: What is the difference between `docker run` and `docker start`?**

- **`docker run`**: Creates a NEW container from an image and starts it
- **`docker start`**: Starts an EXISTING stopped container

```bash
# Creates and starts a new container
docker run -d --name web nginx

# Stops the container
docker stop web

# Starts the existing container
docker start web
```

**Q18: How do you view logs from a container?**

```bash
# View all logs
docker logs container-name

# Follow logs (like tail -f)
docker logs -f container-name

# Show last 100 lines
docker logs --tail 100 container-name

# Show logs with timestamps
docker logs -t container-name
```

**Q19: What is the difference between `docker exec` and `docker attach`?**

- **`docker exec`**: Run a NEW command in a running container (doesn't attach to main process)
- **`docker attach`**: Attach to the MAIN process (PID 1) of the container

```bash
# Execute bash in running container (recommended)
docker exec -it container-name bash

# Attach to main process (Ctrl+C will stop container!)
docker attach container-name
```

**Q20: What are container health checks?**

Health checks periodically test if a container is working correctly. Docker can automatically restart unhealthy containers.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```yaml
# docker-compose.yml
services:
  web:
    image: my-app
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
```

**Health States:**

- `starting`: Initial state
- `healthy`: Health check passed
- `unhealthy`: Health check failed

**Q21: What is container orchestration?**

Container orchestration automates the deployment, scaling, networking, and management of containerized applications across a cluster of machines.

**Popular Orchestration Tools:**

- **Kubernetes**: Industry standard, most feature-rich
- **Docker Swarm**: Simpler, integrated with Docker
- **ECS/EKS**: AWS-managed solutions
- **Nomad**: HashiCorp's orchestrator

**Q22: What are the security best practices for containers?**

1. **Use official/trusted base images**: Reduce vulnerabilities
2. **Don't run as root**: Use `USER` instruction in Dockerfile
3. **Scan images for vulnerabilities**: Use tools like Trivy, Snyk
4. **Keep images updated**: Regularly rebuild with latest base images
5. **Use secrets management**: Don't hardcode credentials
6. **Limit resources**: Use CPU/memory limits
7. **Use read-only filesystems**: When possible

```dockerfile
# Create non-root user
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs
WORKDIR /app
COPY --chown=nodejs:nodejs . .
CMD ["node", "server.js"]
```

**Q23: What is the difference between `docker stop` and `docker kill`?**

- **`docker stop`**: Sends SIGTERM (graceful shutdown), waits 10s, then SIGKILL
- **`docker kill`**: Immediately sends SIGKILL (forceful termination)

Always use `docker stop` to allow cleanup (database connections, save state, etc.).

**Q24: What is a .dockerignore file?**

Similar to `.gitignore`, it prevents files from being copied into the Docker image during build. This reduces image size and build time.

```dockerignore
node_modules
npm-debug.log
.git
.env
*.md
.github
```

**Q25: What happens when you run `docker run` behind the scenes?**

1. **Image Check**: Docker checks if the image exists locally
2. **Pull**: If not found locally, pulls from registry (Docker Hub by default)
3. **Container Creation**: Creates a new container from the image
4. **Filesystem**: Allocates a read-write filesystem layer on top of the image
5. **Network**: Creates a network interface and assigns an IP
6. **Process Start**: Executes the command specified in CMD/ENTRYPOINT
7. **Streams**: Captures and forwards STDOUT/STDERR

**Q26: What are the different container states?**

- **created**: Container created but not started
- **running**: Container is running
- **paused**: Container processes are paused (frozen)
- **restarting**: Container is in the process of restarting
- **exited**: Container stopped (exit code 0 = success, non-zero = error)
- **dead**: Container failed to stop/remove (rare)

```bash
# View container states
docker ps -a
```

**Q27: How do you limit container resources?**

Prevent containers from consuming all host resources:

```bash
# Limit memory to 512MB
docker run -m 512m nginx

# Limit CPU (50% of 1 core)
docker run --cpus="0.5" nginx

# Limit CPU (relative weight vs other containers)
docker run --cpu-shares=512 nginx

# Combine limits
docker run -m 512m --cpus="0.5" nginx
```

**docker-compose.yml:**

```yaml
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 256M
```
