## 🏗️ Docker Architecture

Docker architecture consists of several key components, each playing a vital role in the containerization process.

![Docker Architecture](https://github.com/HarishDubbaka/Docker-k8s/blob/2a40ff1c1827fd4923216fb3f519865feea0484e/Day02/ChatGPT%20Image%20Jan%2017%2C%202026%2C%2006_50_55%20AM.png)

---

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

2. The Docker daemon either **creates a new image** or **pulls an existing image** from the Docker registry.

3. The Docker daemon then **creates an instance of the Docker image**.

4. When the client issues a `docker run` command, the daemon **creates and starts a container** from the image.

This represents a simple and typical Docker workflow.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/2c9e00a23e71cc8686ac66ae993e636c58b60432/Day02/ChatGPT%20Image%20Jan%2017%2C%202026%2C%2010_51_06%20AM.png)

---

## ⚙️ Docker Engine

Docker Engine is the **core component** of the Docker platform. It enables containerization using a **client-server architecture**.

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

- Contains pre-installed software, libraries, and dependencies  
- Portable and reusable  
- Can be pulled from Docker Hub instead of building from scratch  

Docker images are created using **Dockerfiles**, which contain step-by-step instructions for building the image layer by layer.

---

## 🧱 Docker Container

Docker containers are the **running instances of Docker images**.

- One image can create multiple containers  
- Lightweight, isolated, and fast  
- Packages application code, dependencies, and binaries together  

Like cargo containers, Docker containers can be built, shipped, run, stopped, modified, or removed independently.

---

## 🗂️ Docker Registry

A Docker registry is a **repository used to store Docker images**.

- **Public Registry:** Docker Hub  
- **Private Registry:** Used by organizations to store proprietary images securely  

Registries make it easy to share, version, and deploy container images across environments.

---

## ✅ Verify Docker Desktop Installation

Docker Desktop is an all-in-one application used to build Docker images, run containers, and manage Docker environments.

Before proceeding, make sure Docker Desktop is **installed and running** on your system.

![Docker Desktop](https://github.com/HarishDubbaka/Docker-k8s/blob/21e3c93c74af1b46513b187d4d156d756d129d31/Day02/docker%20desktop.png)

---

### 🔍 How to Verify Installation

1. Open a terminal or command prompt.
2. Run the following command:

```bash
docker --version
````

If Docker is installed correctly, the version details will be displayed.

💡 Ensure Docker Desktop is running (Docker 🐳 icon visible in the system tray).

---

## 🚀 Run Your First Docker Container

Once Docker Desktop is installed and set up, you’re ready to run a container.

### ▶️ Try It Out

#### 1️⃣ Start a Container

```bash
docker run -d -p 8080:80 docker/welcome-to-docker
```

#### 2️⃣ Access the Frontend

This container exposes the application on port **8080**.

🌐 Open your browser and visit:

```
http://localhost:8080
```

You should now see the **Welcome to Docker** page running successfully 🎉

✅ Congratulations! You’ve successfully verified Docker Desktop and run your first Docker container.

![First Container](https://github.com/HarishDubbaka/Docker-k8s/blob/5ef0ed0a037751fc2dc70e6b1fe43b6894d55893/Day02/first%20container.png)


---
