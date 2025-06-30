# Complete Dockerfile Creation Guide

## 📋 Table of Contents
1. [Dockerfile Basics](#dockerfile-basics)
2. [Step-by-Step Creation Process](#step-by-step-creation-process)
3. [Language-Specific Examples](#language-specific-examples)
4. [Multi-Stage Builds](#multi-stage-builds)
5. [Best Practices](#best-practices)
6. [Common Patterns](#common-patterns)
7. [Troubleshooting](#troubleshooting)

---

## 🏗️ Dockerfile Basics

### What is a Dockerfile?
A Dockerfile is a text file containing instructions to build a Docker image. Each instruction creates a new layer in the image.

### Basic Dockerfile Structure
```dockerfile
# Comment
INSTRUCTION arguments
```

### Essential Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:18-alpine` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files from host | `COPY . .` |
| `RUN` | Execute commands | `RUN npm install` |
| `EXPOSE` | Document port | `EXPOSE 3000` |
| `CMD` | Default command | `CMD ["npm", "start"]` |
| `ENV` | Set environment variables | `ENV NODE_ENV=production` |

---

## 🚀 Step-by-Step Creation Process

### Step 1: Analyze Your Project
```bash
# Ask yourself these questions:
# 1. What runtime/language does my app use?
# 2. What dependencies does it need?
# 3. How does it start?
# 4. What ports does it use?
# 5. Does it need any environment variables?
```

### Step 2: Choose Base Image
```dockerfile
# Option 1: Official language image
FROM node:18-alpine

# Option 2: Minimal OS image
FROM alpine:3.18

# Option 3: Distroless image (for production)
FROM gcr.io/distroless/nodejs18-debian11
```

### Step 3: Set Working Directory
```dockerfile
# Create and set working directory
WORKDIR /app
```

### Step 4: Copy Dependencies First (for caching)
```dockerfile
# Copy package files first for better caching
COPY package*.json ./
RUN npm ci --only=production
```

### Step 5: Copy Application Code
```dockerfile
# Copy rest of the application
COPY . .
```

### Step 6: Set Runtime Configuration
```dockerfile
# Expose port (documentation only)
EXPOSE 3000

# Set default command
CMD ["npm", "start"]
```

---

## 🌐 Language-Specific Examples

### Node.js Application
```dockerfile
# Use official Node.js runtime
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Copy application code
COPY --chown=nextjs:nodejs . .

# Expose port
EXPOSE 3000

# Switch to non-root user
USER nextjs

# Start application
CMD ["npm", "start"]
```

### Python Application
```dockerfile
# Use official Python runtime
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash app && \
    chown -R app:app /app
USER app

# Expose port
EXPOSE 8000

# Run application
CMD ["python", "app.py"]
```

### Java Spring Boot Application
```dockerfile
# Build stage
FROM maven:3.8.6-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:17-jdk-slim
WORKDIR /app

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Create non-root user
RUN groupadd -r spring && useradd -r -g spring spring
USER spring

# Expose port
EXPOSE 8080

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Go Application
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Runtime stage
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/

# Copy binary from builder stage
COPY --from=builder /app/main .

# Expose port
EXPOSE 8080

# Run application
CMD ["./main"]
```

### React Application (with Nginx)
```dockerfile
# Build stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🎯 Multi-Stage Builds

### Why Use Multi-Stage Builds?
- Smaller final images
- Separate build and runtime environments
- Better security (no build tools in production)

### Advanced Multi-Stage Example
```dockerfile
# Stage 1: Base dependencies
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Development dependencies
FROM base AS dev-deps
RUN npm ci

# Stage 3: Build application
FROM dev-deps AS build
COPY . .
RUN npm run build
RUN npm run test

# Stage 4: Production image
FROM node:18-alpine AS production
WORKDIR /app

# Copy production dependencies
COPY --from=base /app/node_modules ./node_modules

# Copy built application
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodeuser -u 1001

# Set ownership
RUN chown -R nodeuser:nodejs /app
USER nodeuser

EXPOSE 3000
CMD ["npm", "start"]
```

---

## ✅ Best Practices

### 1. Layer Optimization
```dockerfile
# ❌ Bad: Creates multiple layers
RUN apt-get update
RUN apt-get install -y git
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# ✅ Good: Single layer
RUN apt-get update && \
    apt-get install -y git curl && \
    rm -rf /var/lib/apt/lists/*
```

### 2. Use .dockerignore
```dockerignore
# .dockerignore file
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
```

### 3. Leverage Build Cache
```dockerfile
# ✅ Copy dependencies first (changes less frequently)
COPY package*.json ./
RUN npm install

# ✅ Copy source code last (changes more frequently)
COPY . .
```

### 4. Use Specific Tags
```dockerfile
# ❌ Avoid latest tag
FROM node:latest

# ✅ Use specific versions
FROM node:18.17.0-alpine
```

### 5. Run as Non-Root User
```dockerfile
# Create and use non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser
```

### 6. Health Checks
```dockerfile
# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

---

## 🔧 Common Patterns

### Environment Configuration
```dockerfile
# Use build arguments for flexibility
ARG NODE_ENV=production
ENV NODE_ENV=$NODE_ENV

# Set multiple environment variables
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info
```

### Secret Management
```dockerfile
# Use build secrets (BuildKit)
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=mypassword \
  cat /run/secrets/mypassword
```

### Volume Management
```dockerfile
# Create volume for persistent data
VOLUME ["/app/data"]

# Set proper permissions
RUN mkdir -p /app/data && \
    chown -R appuser:appuser /app/data
```

### Conditional Logic
```dockerfile
# Install development dependencies conditionally
ARG NODE_ENV=production
RUN if [ "$NODE_ENV" = "development" ]; then npm install; else npm ci --only=production; fi
```

---

## 🛠️ Building and Testing

### Build Your Dockerfile
```bash
# Basic build
docker build -t myapp:latest .

# Build with build arguments
docker build --build-arg NODE_ENV=development -t myapp:dev .

# Build with custom Dockerfile name
docker build -f Dockerfile.prod -t myapp:prod .

# Build and push
docker build -t myregistry/myapp:v1.0 .
docker push myregistry/myapp:v1.0
```

### Test Your Image
```bash
# Run and test
docker run -d --name test-container -p 3000:3000 myapp:latest

# Check logs
docker logs test-container

# Execute commands inside
docker exec -it test-container sh

# Check resource usage
docker stats test-container

# Clean up
docker stop test-container && docker rm test-container
```

---

## 🐛 Troubleshooting

### Common Issues and Solutions

#### 1. Large Image Size
```dockerfile
# ❌ Problem: Large base image
FROM ubuntu:latest

# ✅ Solution: Use alpine or slim variants
FROM node:18-alpine
```

#### 2. Build Context Too Large
```bash
# Check build context size
docker build --no-cache .

# Solution: Use .dockerignore
echo "node_modules" >> .dockerignore
echo ".git" >> .dockerignore
```

#### 3. Permission Issues
```dockerfile
# ❌ Problem: Running as root
USER root

# ✅ Solution: Create and use non-root user
RUN adduser -D appuser
USER appuser
```

#### 4. Cache Not Working
```dockerfile
# ❌ Problem: Dependencies copied with source
COPY . .
RUN npm install

# ✅ Solution: Copy dependencies first
COPY package*.json ./
RUN npm install
COPY . .
```

---

## 📝 Dockerfile Templates

### Basic Web Application
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN addgroup -g 1001 -S nodejs && adduser -S nodeuser -u 1001
USER nodeuser
EXPOSE 3000
CMD ["npm", "start"]
```

### Microservice with Health Check
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd --create-home --shell /bin/bash app && chown -R app:app /app
USER app
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
CMD ["python", "app.py"]
```

---

## 🎓 Next Steps

1. **Learn Docker Compose** - For multi-container applications
2. **Explore BuildKit** - For advanced build features
3. **Study Security Scanning** - Use tools like `docker scout`
4. **Practice with CI/CD** - Integrate with GitHub Actions or GitLab CI
5. **Optimize for Production** - Learn about distroless images and security hardening

---

## 📚 Additional Resources

- [Official Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Multi-stage Builds](https://docs.docker.com/develop/dev-best-practices/#use-multi-stage-builds)

Remember: Start simple, then optimize. A working Dockerfile is better than a perfect one that doesn't exist! 🚀