## 🏗️ Docker Architecture

Docker architecture consists of several key components, each playing a vital role in the containerization process.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/2a40ff1c1827fd4923216fb3f519865feea0484e/Day02/ChatGPT%20Image%20Jan%2017%2C%202026%2C%2006_50_55%20AM.png)

### 🔹 Core Components

- **Docker Client**  
  The Docker client is used by users to interact with Docker. It sends commands to the Docker Host.

- **Docker Host**  
  The Docker Host runs the Docker daemon and is responsible for managing Docker images and containers.

- **Docker Daemon (dockerd)**  
  The Docker daemon runs inside the Docker Host and handles building, running, and managing Docker images and containers.

- **Docker Registry**  
  A Docker registry stores Docker images. Docker Hub is a public registry, but private registries can also be used.

---

## 🔄 Docker Workflow

The following steps describe how Docker components communicate with each other:

1. The **Docker CLI (Client)** issues a `docker build` command to the Docker Host.  
   The Docker daemon builds an image based on the instructions provided and stores it in a Docker registry (local or Docker Hub).

2. The Docker daemon can either **create a new image** or **pull an existing image** from the Docker registry.

3. The Docker daemon then **creates an instance of the Docker image**.

4. When the client issues a `docker run` command, the daemon **creates and starts a container** from the image.

This represents a simple and typical Docker workflow.

---

## ⚙️ Docker Engine

Docker Engine is the **core component** of the Docker platform. It is open-source and enables containerization using a **client-server architecture**.

Docker Engine includes:

- **Docker Daemon**  
  A background service responsible for managing images, containers, volumes, and networks.

- **Docker CLI (Client)**  
  A command-line interface used to communicate with the Docker daemon.

- **REST API**  
  Allows the Docker CLI and daemon to communicate and exchange instructions.

---

## 📦 Docker Image

A Docker image is a **template used to create containers**.

- Images contain pre-installed software, libraries, and dependencies.
- Images are portable and reusable.
- You can use existing images from Docker Hub instead of building from scratch.

Docker images are created using **Dockerfiles**, which contain step-by-step instructions for building the image layer by layer.

---

## 🧱 Docker Container

Docker containers are the **running instances of Docker images**.

- One image can create multiple containers.
- Containers are lightweight, isolated, and fast.
- Containers package application code, dependencies, and binaries together.

Docker containers are similar to cargo containers:
you can build, ship, run, stop, modify, or remove them easily without affecting other containers.

---

## 🗂️ Docker Registry

A Docker registry is a **repository for storing Docker images**.

- **Public Registry:** Docker Hub  
- **Private Registry:** Used by organizations to store proprietary images securely

Registries make it easy to share, version, and deploy container images across environments.

---
