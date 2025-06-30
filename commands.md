# Docker Commands Reference & Best Practices Guide

## Syntax

```bash
docker [OPTIONS] COMMAND
```

*A self-sufficient runtime for containers*

---

## 🚀 Most Used Commands (Start Here!)

| Command | Description | Example Usage | 💡 Pro Tips |
|---------|-------------|---------------|-------------|
| `docker run` | Create and run a new container | `docker run -d -p 8080:80 nginx` | Use `-d` for detached mode, `-p` for port mapping |
| `docker ps` | List running containers | `docker ps -a` | Add `-a` to see all containers (including stopped) |
| `docker build` | Build an image from Dockerfile | `docker build -t myapp:v1.0 .` | Always tag your images with meaningful names |
| `docker exec` | Execute commands in running container | `docker exec -it container_name bash` | Use `-it` for interactive terminal |
| `docker logs` | View container logs | `docker logs -f container_name` | Use `-f` to follow logs in real-time |
| `docker stop` | Stop running containers | `docker stop $(docker ps -q)` | Stop all containers at once |

---

## 📦 Image Management

| Command | Description | Example | When to Use |
|---------|-------------|---------|-------------|
| `docker pull` | Download image from registry | `docker pull node:18-alpine` | Get latest versions or specific tags |
| `docker push` | Upload image to registry | `docker push myrepo/myapp:latest` | Deploy your custom images |
| `docker images` | List local images | `docker images --filter dangling=true` | Clean up unused images |
| `docker rmi` | Remove images | `docker rmi $(docker images -q)` | Free up disk space |
| `docker tag` | Tag an image | `docker tag myapp:v1 myapp:latest` | Create aliases for images |

### 💡 Image Best Practices
```bash
# Use multi-stage builds to reduce image size
FROM node:18-alpine AS build
# ... build steps
FROM nginx:alpine AS production
COPY --from=build /app/dist /usr/share/nginx/html

# Always specify exact versions in production
docker pull node:18.17.0-alpine  # ✅ Good
docker pull node:latest          # ❌ Avoid in production
```

---

## 🔧 Container Operations

| Command | Description | Example | Common Use Cases |
|---------|-------------|---------|------------------|
| `docker create` | Create container without starting | `docker create --name myapp nginx` | Prepare containers for later use |
| `docker start` | Start stopped containers | `docker start myapp` | Resume paused work |
| `docker restart` | Restart containers | `docker restart myapp` | Apply configuration changes |
| `docker rm` | Remove containers | `docker rm -f $(docker ps -aq)` | Clean up all containers |
| `docker cp` | Copy files to/from containers | `docker cp file.txt myapp:/tmp/` | Transfer files without volumes |

### 🎯 Container Lifecycle Tips
```bash
# One-liner to stop and remove all containers
docker stop $(docker ps -q) && docker rm $(docker ps -aq)

# Run temporary containers that auto-remove
docker run --rm -it ubuntu:latest bash

# Run with resource limits
docker run -m 512m --cpus=1.5 myapp
```

---

## 🌐 Networking & Volumes

| Command | Description | Example | Use Case |
|---------|-------------|---------|----------|
| `docker network create` | Create custom network | `docker network create mynetwork` | Connect multiple containers |
| `docker volume create` | Create persistent storage | `docker volume create mydata` | Persist data between container restarts |
| `docker port` | Show port mappings | `docker port container_name` | Debug connectivity issues |

### 🔗 Networking Examples
```bash
# Create a custom bridge network
docker network create --driver bridge myapp-network

# Run containers on the same network
docker run -d --network myapp-network --name database postgres
docker run -d --network myapp-network --name web nginx

# Connect existing container to network
docker network connect myapp-network existing-container
```

---

## 🔍 Debugging & Monitoring

| Command | Description | Example | Debugging Scenario |
|---------|-------------|---------|-------------------|
| `docker inspect` | Get detailed container info | `docker inspect container_name` | Check configuration and networking |
| `docker stats` | Live resource usage | `docker stats --no-stream` | Monitor CPU/memory usage |
| `docker events` | Monitor Docker events | `docker events --since '1h'` | Track container lifecycle events |
| `docker diff` | See filesystem changes | `docker diff container_name` | Debug file modifications |
| `docker top` | Show running processes | `docker top container_name` | Check what's running inside |

### 🐛 Common Debugging Commands
```bash
# Check container health and logs
docker ps -a  # See exit codes
docker logs --tail 50 container_name  # Recent logs
docker exec -it container_name sh  # Get inside container

# Troubleshoot networking
docker network ls  # List networks
docker network inspect bridge  # Check network config
docker port container_name  # Check port mappings
```

---

## 🧹 Cleanup & Maintenance

| Command | Description | Example | Frequency |
|---------|-------------|---------|-----------|
| `docker system prune` | Remove unused data | `docker system prune -a` | Weekly |
| `docker image prune` | Remove unused images | `docker image prune --filter "until=24h"` | Daily |
| `docker container prune` | Remove stopped containers | `docker container prune` | Daily |
| `docker volume prune` | Remove unused volumes | `docker volume prune` | Weekly |

### 🧽 Cleanup Scripts
```bash
#!/bin/bash
# Docker cleanup script - save as cleanup-docker.sh

echo "🧹 Cleaning up Docker..."

# Remove stopped containers
docker container prune -f

# Remove unused images
docker image prune -f

# Remove unused volumes (be careful!)
docker volume prune -f

# Remove unused networks
docker network prune -f

# Show disk usage
docker system df
```

---

## 🏗️ Development Workflows

### 🔄 Hot Reload Development
```bash
# Mount source code for live development
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:18-alpine \
  npm run dev
```

### 🚀 Multi-Container Applications
```bash
# Use Docker Compose for complex applications
# docker-compose.yml example:
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - database
  database:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
```

---

## ⚡ Power User Commands

| Command | Description | Example | Advanced Use |
|---------|-------------|---------|--------------|
| `docker buildx` | Advanced build features | `docker buildx build --platform linux/amd64,linux/arm64` | Multi-platform builds |
| `docker context` | Switch Docker environments | `docker context use production` | Manage multiple Docker hosts |
| `docker manifest` | Multi-architecture images | `docker manifest inspect nginx` | Check supported platforms |

---

## 🛡️ Security Best Practices

```bash
# Run containers as non-root user
docker run --user 1000:1000 myapp

# Use read-only containers when possible
docker run --read-only --tmpfs /tmp myapp

# Limit container capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp

# Scan images for vulnerabilities
docker scout cves myimage:latest
```

---

## 📚 Quick Reference

### Essential Aliases (Add to ~/.bashrc)
```bash
alias dps='docker ps'
alias dpa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dlog='docker logs -f'
alias dstop='docker stop $(docker ps -q)'
alias dclean='docker system prune -f'
```

### Environment Variables
```bash
# Common Docker environment variables
DOCKER_HOST=tcp://docker-host:2376
DOCKER_TLS_VERIFY=1
DOCKER_CERT_PATH=/path/to/certs
```

---

## 🆘 Getting Help

```bash
# Get help for any command
docker COMMAND --help

# Get detailed Docker info
docker info

# Check Docker version
docker version

# View system-wide Docker usage
docker system df
```

**💡 Remember:** Always test commands in development before running in production!

---

*This guide covers Docker CLI commands. For orchestration, consider Docker Compose or Kubernetes.*

