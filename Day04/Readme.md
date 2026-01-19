# 🐳 Docker Image Layers and Dockerfile Basics

This document explains **Docker image layers**, **immutability**, **union filesystems**, and how to **create and run a PHP application using Docker**, with clear ASCII diagrams for better understanding 📘

---

## 🔒 What does “immutable image layers” mean?

**Immutable** means **unchangeable**.  

Once a container image layer is created, it **cannot be modified**.  
If changes are required, Docker creates a **new layer on top** instead of editing the existing one.

### 📌 Image Layer Immutability Diagram

```

## Layer 5: Application Code     (immutable)

## Layer 4: Dependencies         (immutable)

## Layer 3: requirements.txt     (immutable)

## Layer 2: Runtime              (immutable)

Layer 1: Base OS              (immutable)

````

✅ Once created, a layer remains the same forever.

---

## 📦 What is inside an image layer?

Each image layer contains **filesystem changes only**, such as:

* ➕ Adding files
* ✏️ Modifying files
* ❌ Deleting files

### 🧱 Example Image Layers

1. Base OS and package manager (`apt`)  
2. Runtime (Python / PHP / Node)  
3. Dependency definition file (`requirements.txt`)  
4. Installed dependencies  
5. Application source code  

Each step creates **one new immutable layer**.

---

## 📝 Example Dockerfile and Layer Creation

```dockerfile
FROM ubuntu:22.04          # Layer 1

RUN apt-get update && \
    apt-get install -y python3 python3-pip   # Layer 2

COPY requirements.txt /app/requirements.txt  # Layer 3

RUN pip3 install -r /app/requirements.txt    # Layer 4

COPY . /app                                  # Layer 5
````

### 📌 Dockerfile to Image Layers Diagram

```
Dockerfile Instructions
        |
        v
+----------------------+
| FROM ubuntu:22.04    |  -> Layer 1
+----------------------+
| RUN install runtime  |  -> Layer 2
+----------------------+
| COPY requirements    |  -> Layer 3
+----------------------+
| RUN install deps     |  -> Layer 4
+----------------------+
| COPY app source      |  -> Layer 5
+----------------------+
```

---

## 🗂️ How do layers become a filesystem?

Docker uses a **union filesystem** to stack layers together.

### 📌 Union Filesystem Diagram

```
Merged Container Filesystem (/)
================================
Application Code
--------------------------------
Dependencies
--------------------------------
Runtime
--------------------------------
Base OS
```

👉 Although layers are stored separately on disk, the container sees **one unified filesystem**.

---

## ✍️ Writing files inside a container

Image layers are **read-only**, so Docker adds a **writable layer** when a container starts.

### 📌 Container Writable Layer Diagram

```
Writable Container Layer (runtime changes)
========================================
Image Layer: Application Code
----------------------------------------
Image Layer: Dependencies
----------------------------------------
Image Layer: Runtime
----------------------------------------
Image Layer: Base OS
```

* All file changes go into the writable layer
* When the container is deleted:

  * ❌ Writable layer is removed
  * ✅ Image layers are reused

---

## 📄 Dockerfile Overview

A **Dockerfile** is a text file containing instructions to build a Docker image.

* 🧱 Each instruction creates a new layer
* 📦 Layers stack to form the final image
* 🔨 Images are built using `docker build`
* ▶️ Containers are started using `docker run`

---

## 📋 Common Dockerfile Instructions

| Instruction | Purpose                    |
| ----------- | -------------------------- |
| FROM        | Defines base image         |
| RUN         | Executes commands          |
| COPY        | Copies files               |
| ADD         | Copies with extra features |
| ENV         | Sets environment variables |
| WORKDIR     | Sets working directory     |
| EXPOSE      | Documents ports            |
| CMD         | Default container command  |
| ENTRYPOINT  | Main application           |

---

## 🚀 Creating and Running a Sample PHP Application

### Step 1️⃣ Create a Dockerfile

Create a file named **`Dockerfile`** (no extension):

```dockerfile
FROM php:8.0-apache

# Set working directory
WORKDIR /var/www/html

# Copy application source code
COPY ./src/ .

# Enable Apache rewrite module
RUN a2enmod rewrite

# Expose port 80
EXPOSE 80

# Start Apache server
CMD ["apache2-foreground"]
```

---

### 📂 Required Project Structure

```
project-folder/
├── Dockerfile
└── src/
    └── index.php
```

---

### 🧪 Sample `index.php`

```php
<?php
echo "Hello World from Docker! 🚀";
```

---

### 🔨 Build the Docker Image

```bash
docker build -t sample-php-app .
```

---

### ▶️ Run the Docker Container

```bash
docker run -p 8000:80 sample-php-app
```

---

### 🌐 Access the Application

Open your browser and visit:

```
http://localhost:8000
```

---

## 📤 Push Your Image to Docker Hub

### 1️⃣ Create a Docker Hub Account

Go to [Docker Hub](https://hub.docker.com/) and sign up.
You’ll need a **username** (e.g., `yourusername`) for tagging images.

---

### 2️⃣ Login to Docker Hub

```bash
docker login
```

Enter your Docker Hub username and password.
You should see:

```
Login Succeeded
```

---

### 3️⃣ Tag the Image

```bash
docker tag sample-php-app yourusername/sample-php-app:latest
```

* `yourusername` → Docker Hub username
* `sample-php-app` → repository name
* `latest` → tag (optional)

Verify with:

```bash
docker images
```

---

### 4️⃣ Push the Image

```bash
docker push yourusername/sample-php-app:latest
```

Docker will upload all layers.
Once done, the image is available on Docker Hub.

---

### 5️⃣ Verify and Pull

On Docker Hub, check your repository.
Anyone can pull the image with:

```bash
docker pull yourusername/sample-php-app:latest
```

---

### ✅ Quick Summary

1. `docker login` → Login to Docker Hub
2. `docker tag <local-image> <username>/<repo>:<tag>` → Tag image
3. `docker push <username>/<repo>:<tag>` → Push image
4. Verify and share the image

---

## 🧠 What This Dockerfile Does

| Instruction | Description                  |
| ----------- | ---------------------------- |
| FROM        | Uses PHP + Apache base image |
| WORKDIR     | Sets working directory       |
| COPY        | Copies application files     |
| RUN         | Enables Apache rewrite       |
| EXPOSE      | Documents port 80            |
| CMD         | Starts Apache server         |

---

## ✅ Summary

* 🧩 Docker images are built from **immutable layers**
* 🧱 Containers add a **writable layer**
* 📁 Union filesystem merges layers
* 📝 Dockerfile defines image creation
* 🌍 Images run consistently in any environment

---

🎉 **Happy Dockering!** 🐳🚀



