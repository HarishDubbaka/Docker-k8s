# 🚀 Mastering Multi-Stage Docker Builds for a React App

A practical guide to multi-stage Docker builds 🐳, showing how to create lightweight, optimized, and production-ready container images.
This project highlights how separating the build environment from the runtime environment can dramatically shrink your final image size while improving efficiency

---

## 📋 Table of Contents
- Introduction
- Why Multi-Stage Builds?
- Prerequisites
- Step-by-Step Tutorial
- How It Works: Deconstructing the Dockerfile
- Inspecting the Final Container
- What Happens if You Delete `index.html`
- Restoring Deleted Files
- Useful Docker Commands
- Best Practices
- Resources

---

## 🧩 Introduction
Welcome! 👋  
This project provides a hands-on example of a **multi-stage Docker build**. The goal is to take a standard React application, which requires a Node.js environment with many dependencies to build, and package it into a **super lightweight Nginx container** for production.

By following this guide, you'll learn one of the most effective techniques for optimizing your Docker images, leading to:

- ⚡ Faster deployments  
- 🔐 Improved security  
- 📦 Smaller image sizes  
- 🚀 Better runtime performance  

---

## ❓ Why Multi-Stage Builds?
The primary reason is **image size optimization** 📉.

A typical image may contain:
- Base OS  
- Node.js runtime  
- All `node_modules` dependencies  
- Full source code  

But for a **production frontend app**, you only need:
- Static files (HTML, CSS, JavaScript)  
- A web server 🌐  

### Multi-stage builds solve this by:
- 🧱 **Separating concerns**: One stage for building, another for running  
- 📦 **Reducing image size**: No build-time dependencies in production  
- 🔐 **Improving security**: Smaller attack surface  
- ⚡ **Boosting performance**: Faster pulls, pushes, and deployments  

---

## 🛠️ Prerequisites
- 🐳 Docker  
- 🌱 Git  

---

## 🪜 Step-by-Step Tutorial

### Step 1: Clone the Project Repository
```bash
git clone https://github.com/piyushsachdeva/todoapp-docker.git
cd todoapp-docker
```

### Step 2: Create the Dockerfile
```bash
touch Dockerfile
```

### Step 3: Add the Multi-Stage Build Instructions
```dockerfile
# Stage 1: Installer / Builder
FROM node:18-alpine AS installer
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Deployer / Runner
FROM nginx:latest AS deployer
COPY --from=installer /app/build /usr/share/nginx/html
EXPOSE 80
```

### Step 4: Build the Docker Image
```bash
docker build -t multi-stage .
```

### Step 5: Run the Docker Container
```bash
docker run -d -p 3000:80 multi-stage
```
Visit 👉 `http://localhost:3000` [(localhost in Bing)](https://www.bing.com/search?q="http%3A%2F%2Flocalhost%3A3000%2F")

---

## 🔍 How It Works: Deconstructing the Dockerfile

### Stage 1: Installer 🏗️
- Uses Node.js to install dependencies  
- Builds the React app into static files  
- Outputs everything into `/app/build`  

### Stage 2: Deployer 🚢
- Starts with a clean Nginx image  
- Copies only the built assets from the previous stage  
- No Node.js, no source code, no `node_modules` 🙌  

Result: **tiny, production-ready image** optimized for serving static files.

---

## 🔍 Inspecting the Final Container
```bash
docker ps
docker exec -it <CONTAINER_ID> sh
cd /usr/share/nginx/html
ls
```

Expected output:
```
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

---

## ❌ What Happens if You Delete `index.html`?
- File disappears from that container’s writable layer.  
- Restarting the container does **not** restore it.  
- Recreating the container from the image restores it (image is immutable).  

### ⚖️ Analogy
- **Image = recipe** (unchanged)  
- **Container = dish** (can be modified, but recipe stays safe)  

---

## 🔄 Restoring Deleted Files
1. **Recreate the container (recommended)**  
   ```bash
   docker rm -f <CONTAINER_ID>
   docker run -d -p 3000:3000 multi-stage
   ```

2. **Copy file back**  
   ```bash
   docker cp index.html <CONTAINER_ID>:/usr/share/nginx/html/index.html
   ```

3. **Rebuild the image**  
   ```bash
   docker build -t multi-stage .
   docker run -d -p 3000:3000 multi-stage
   ```

---

## 📚 Useful Docker Commands
| Command | Description |
|---------|-------------|
| `docker build -t <tag> .` | Build an image |
| `docker run -d -p <host>:<cont> <tag>` | Run a container |
| `docker ps` | List running containers |
| `docker images` | List local images |
| `docker exec -it <id> sh` | Open shell in container |
| `docker logs <id>` | Show logs |
| `docker inspect <id>` | Inspect container |
| `docker stop <id>` | Stop container |
| `docker rm <id>` | Remove container |

---

## ✅ Best Practices
- 🧹 Use `.dockerignore` to exclude `node_modules`, `.git`, `.env`  
- 🔐 Run as non-root user  
- 🧠 Leverage layer caching (copy `package.json` first)  
- 📦 Use Alpine-based images for smaller size  

---

## 🔗 Resources
- 📘 Docker Docs: Multi-stage builds [(docs.docker.com in Bing)](https://www.bing.com/search?q="https%3A%2F%2Fdocs.docker.com%2Fdevelop%2Fdevelop-images%2Fmultistage-build%2F")  
- 🛠️ Docker best practices for writing Dockerfiles  

---

✨ Happy Dockerizing! 🐳
```

