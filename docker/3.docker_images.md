# Docker Images

---

# What is a Docker Image?

A Docker image is a **read-only blueprint** used to create containers.

It contains:
- Application code
- Dependencies
- Libraries
- Runtime
- Configuration

    **Image** = Template  
    **Container** = Running instance of image  

Images are:
- Built from a Dockerfile
- Made up of multiple layers

---

# How Images Are Created

Images are built using a **Dockerfile**.

A Dockerfile is a text file containing instructions
that Docker executes from top to bottom.

Build command:

`docker build -t image-name .`


Docker tag:

Creates a new name (tag) for an existing image.  
It does NOT create a new image.

Used for:
- Versioning
- Managing environments

## Syntax

`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`

---

Example

`docker tag myapp:v1 myapp:latest`

Tag for Docker Hub

`docker tag myapp:v1 username/myapp:v1`

Push command:

`docker push username/myapp:v1`

---

# Dockerfile Instructions

---

## FROM

Defines the base image.

Every Dockerfile must start with `FROM.`

Example:

`FROM ubuntu:22.04`

This means:
Start building image using Ubuntu 22.04.

---

## WORKDIR

Sets the working directory inside the image.

Example:

`WORKDIR /app`

If the directory does not exist → Docker creates it.

---

## COPY

Copies files from your local machine into the image.

Syntax:

`COPY <source> <destination>`

Example:

`COPY . .`

Copies all files into the working directory.

---

## ADD

Similar to COPY but:
- Can extract .tar files
- Can download from URL

Example 1: Extract `.tar` File

```bash
FROM ubuntu:22.04
WORKDIR /app
ADD app.tar.gz /app/
```
Docker will:
Copy app.tar.gz & automatically extract it into /app/

Example 2: Download from URL
```bash
FROM ubuntu:22.04
WORKDIR /app
ADD https://example.com/file.txt /app/file.txt
```

Docker will: Download the file & Save it to /app/file.txt
## RUN

Executes commands during image build.

Used for:
- Installing packages
- Building app
- Updating system

Example:

`RUN apt update && apt install -y curl`

Runs only during build.

Each RUN creates a new layer.

---

## ENV

Sets environment variables inside the image.

Example:
```bash
ENV NODE_ENV=production`
```
---

## ARG

Defines build-time variables.

Example:
```bash
ARG VERSION=1.0
```
Used like:

`docker build --build-arg VERSION=2.0 .`

Available only during build.

---

## EXPOSE

Documents which port the container uses.

Example:

`EXPOSE 3000`

---

## CMD

Defines default command when container starts.

Example:
```bash
CMD ["node", "app.js"]
```
Can be overridden:

`docker run image-name echo "Hello"`

---

## ENTRYPOINT

Defines main command that always runs.

Arguments passed at runtime are appended.

Example:
```bash
ENTRYPOINT ["echo"]
```
`docker run image-name Hello`

Output:
Hello

---

## USER

Defines which user runs inside container.

Example:
```bash
USER node
```
Improves security.

---

## LABEL

Adds metadata to image.

Example:
```bash
LABEL version="1.0"
LABEL maintainer="admin@example.com"
```

---

# Custom Image Example (Simple)

Create folder:

mkdir my-first-image  
cd my-first-image  

Create Dockerfile:

```bash
FROM ubuntu:22.04  
RUN apt update && apt install -y curl  
CMD ["echo", "Hello from my custom image!"]
```

Build:

`docker build -t my-ubuntu:v1 .`

Run:

`docker run my-ubuntu:v1`

Output:
`Hello from my custom image!`

---

# Custom Web App Image Example

Create index.html:

<h1>Hello from My Docker Image</h1>

Dockerfile:

```bash
FROM nginx:alpine  
COPY index.html /usr/share/nginx/html/  
EXPOSE 80  
```
Build:

`docker build -t my-website:v1 .`

Run:

`docker run -d -p 8080:80 my-website:v1`

Open:

http://localhost:8080

---

# Image Layers

Each Dockerfile instruction creates a **layer**.

Example:

```bash
FROM ubuntu  
RUN apt install curl  
COPY . .  
```
Each line = one layer.

Layers are:
- Read-only
- Stacked on top of each other

---

# Why Layers Are Important

Layers allow:

- Faster builds
- Shared storage between images
- Faster downloads (only new layers pulled)
- Efficient caching

---

# Docker Build Cache

Docker builds images step by step.

If a step does not change:
→ Docker reuses cached layer.

Example (Good Order):

```bash
COPY package.json .  
RUN npm install  
COPY . .  
```
If only source code changes:
- npm install is NOT executed again
- Build is faster

---

# Why Instruction Order Matters

Bad order:
```bash
COPY . .  
RUN npm install  
```
Any code change rebuilds everything

Good order:
```bash
COPY package.json .  
RUN npm install  
COPY . .  
```
Dependencies rebuild only when needed 

Correct order = Faster builds 

---

# Multi-Stage Builds

## What is Multi-Stage Build?

Allows multiple FROM statements in one Dockerfile.

Stage 1 → Build app  
Stage 2 → Copy only final result  

---

## Why Use Multi-Stage?

- Smaller final image
- No build tools in production
- Better security
- Cleaner image

---

## Example: Node.js Multi-Stage
```bash
# -------- Stage 1 --------
FROM node:18 AS builder

WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# -------- Stage 2 --------
FROM node:18-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist

CMD ["node", "dist/app.js"]
```
Result:
Final image does not contain:
- node_modules (dev)
- build tools
- source code

---

# Distroless Images

## What is Distroless?

Distroless images contain:
- Only application
- Only runtime
- No shell
- No package manager
- No extra OS tools

Example base image:

`gcr.io/distroless/nodejs18`

---

## Why Use Distroless?

- Smaller size
- More secure
- Reduced attack surface
- Production ready

You cannot run bash inside distroless.

---

## Example: Multi-Stage + Distroless
```bash
# -------- Stage 1 --------
FROM node:18 AS builder

WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# -------- Stage 2 --------
FROM gcr.io/distroless/nodejs18

WORKDIR /app
COPY --from=builder /app/dist ./dist

CMD ["dist/app.js"]
```
Final image:
- Very small
- No shell
- No npm
- Secure for production

---

#  References

## Multi-Stage Builds

📖 Official Documentation:

https://docs.docker.com/build/building/multi-stage/

---

## Build Cache & Optimization

📖 Docker Build Cache:

https://docs.docker.com/build/cache/

📖 Dockerfile Best Practices:

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/


---

## Distroless Images
📖 Official Distroless Project:

https://github.com/GoogleContainerTools/distroless


---
