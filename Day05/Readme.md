# 🚀 Mastering Multi-Stage Docker Builds for a React App

A concise guide and example of using **multi-stage builds in Docker** to create lean, optimized, and production-ready container images 🐳. This project demonstrates how to significantly reduce your final image size by separating the build environment from the runtime environment.

---

## 📋 Table of Contents

* Introduction
* Why Multi-Stage Builds?
* Prerequisites
* Step-by-Step Tutorial
* How It Works: Deconstructing the Dockerfile
* Inspecting the Final Container
* Useful Docker Commands
* Best Practices
* Resources

---

## 🧩 Introduction

Welcome! 👋
This project provides a hands-on example of a **multi-stage Docker build**. The goal is to take a standard React application, which requires a Node.js environment with many dependencies to build, and package it into a **super lightweight Nginx container** for production.

By following this guide, you'll learn one of the most effective techniques for optimizing your Docker images, leading to:

* ⚡ Faster deployments
* 🔐 Improved security
* 📦 Smaller image sizes
* 🚀 Better runtime performance

---

## ❓ Why Multi-Stage Builds?

So, why go through the trouble of a multi-stage build? The primary reason is **image size optimization** 📉.

Imagine you have a final Docker image that's over **200MB**. This image likely contains:

* A base OS
* Node.js runtime
* All `node_modules` dependencies
* Your full source code

But for a **production frontend app**, you only need:

* Static files (HTML, CSS, JavaScript)
* A web server to serve them 🌐

### Multi-stage builds solve this by:

* 🧱 **Separating concerns**: One stage for building, another for running
* 📦 **Reducing image size**: No build-time dependencies in production
* 🔐 **Improving security**: Smaller attack surface
* ⚡ **Boosting performance**: Faster pulls, pushes, and deployments

---

## 🛠️ Prerequisites

Before you begin, make sure you have the following installed:

* 🐳 Docker
* 🌱 Git

---

## 🪜 Step-by-Step Tutorial

### Step 1: Clone the Project Repository

Clone the sample React application from GitHub:

```bash
git clone https://github.com/piyushsachdeva/todoapp-docker.git
cd todoapp-docker
```

---

### Step 2: Create the Dockerfile

Inside the project directory, create a new file named `Dockerfile`:

```bash
touch Dockerfile
```

---

### Step 3: Add the Multi-Stage Build Instructions

Open the `Dockerfile` and paste the following code 🧑‍💻:

```dockerfile
# Stage 1: The "Installer" or "Builder" Stage
FROM node:18-alpine AS installer

WORKDIR /app

# Copy dependency files first for better caching
COPY package*.json ./

RUN npm install

# Copy the rest of the source code
COPY . .

# Build the React app
RUN npm run build

# Stage 2: The "Deployer" or "Runner" Stage
FROM nginx:latest AS deployer

# Copy built assets from the installer stage
COPY --from=installer /app/build /usr/share/nginx/html

# Expose Nginx's default port
EXPOSE 80
```

---

### Step 4: Build the Docker Image

Build the image and tag it as `multi-stage` 🏗️:

```bash
docker build -t multi-stage .
```

Docker will pull both `node:18-alpine` and `nginx:latest`, but the **final image** will only be based on Nginx ✅.

---

### Step 5: Run the Docker Container

Run the container and map port `3000` on your host to port `80` inside the container:

```bash
docker run -d -p 3000:80 multi-stage
```

Now open your browser and visit 👉 **[http://localhost:3000](http://localhost:3000)**
Your app should be running! 🎉🎉

---

## 🔍 How It Works: Deconstructing the Dockerfile

### Stage 1: The Installer Stage 🏗️

```dockerfile
FROM node:18-alpine AS installer
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
```

This is the **build environment**:

* Uses Node.js to install dependencies 📦
* Builds the React app into static files
* Outputs everything into `/app/build`

---

### Stage 2: The Deployer Stage 🚢

```dockerfile
FROM nginx:latest AS deployer
COPY --from=installer /app/build /usr/share/nginx/html
```

This is the **production environment**:

* Starts with a clean Nginx image 🧼
* Copies only the built assets from the previous stage
* No Node.js, no source code, no `node_modules` 🙌

The result is a **tiny, production-ready image** optimized for serving static files.

---

## 🔍 Inspecting the Final Container

Let’s confirm how lean the container is 🔎.

1. Find your container ID:

```bash
docker ps
```

2. Open a shell inside the container:

```bash
docker exec -it <YOUR_CONTAINER_ID> sh
```

3. Navigate to the web root:

```bash
cd /usr/share/nginx/html
ls
```

You should see output like this 👇:

```text
50x.html
asset-manifest.json
favicon.ico
index.html
logo192.png
logo512.png
manifest.json
robots.txt
static
```

No `node_modules`, no `src` folder 🚫.
That’s multi-stage builds working perfectly! ✅

---

## 📚 Useful Docker Commands

| Command                                | Description                       |
| -------------------------------------- | --------------------------------- |
| `docker build -t <tag> .`              | Builds an image from a Dockerfile |
| `docker run -d -p <host>:<cont> <tag>` | Runs a container in detached mode |
| `docker ps`                            | Lists running containers          |
| `docker images`                        | Lists local Docker images         |
| `docker exec -it <id> sh`              | Opens a shell in a container      |
| `docker logs <id>`                     | Shows container logs              |
| `docker inspect <id>`                  | Displays detailed container info  |
| `docker stop <id>`                     | Stops a running container         |
| `docker rm <id>`                       | Removes a container               |

---

## ✅ Best Practices

* 🧹 **Use a `.dockerignore` file** to exclude `node_modules`, `.git`, and `.env`
* 🔐 **Run as a non-root user** for better security
* 🧠 **Leverage layer caching** by copying `package.json` first
* 📦 **Keep images small** by using Alpine-based images

---

## 🔗 Resources

* 📘 Official Docker Docs: *Multi-stage builds*
* 🛠️ Docker best practices for writing Dockerfiles

---

Happy Dockerizing! 🐳✨
