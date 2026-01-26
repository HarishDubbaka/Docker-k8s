## 👋 About Me

Hi, I’m **Harish Dubbaka** 👨‍💻, an **SAP BASIS professional** passionate about **learning DevOps and cloud-native technologies** ☁️.  
I focus on **Docker 🐳, Kubernetes ☸️, automation, and reliable system operations**, documenting my learning through hands-on practice 📘🛠️.

---

## 🏆 Certifications & Licenses

🎓 **Microsoft Certified: Azure Fundamentals**

🎓 **Microsoft Certified: Azure for SAP Workloads Specialty**

---

## 🛡️ Badges

🟢 **Kyndryl Agile Explorer** – Agile Academy Knowledge Badge

🟢 **Designing and Implementing Microsoft DevOps Solutions**

---

## 🔧 Skills & Tech Stack

![Docker](https://img.shields.io/badge/Docker-Containerization-blue?logo=docker\&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-blue?logo=kubernetes\&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-Cloud%20Platform-blue?logo=microsoftazure\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-OS-black?logo=linux)
![Git](https://img.shields.io/badge/Git-Version%20Control-orange?logo=git)
![DevOps](https://img.shields.io/badge/DevOps-Automation-green)

---

## 🚀 Docker & Kubernetes

🐳 **Docker** packages applications and their dependencies into portable containers.

☸️ **Kubernetes** orchestrates and manages these containers for **scalability, reliability, and self-healing**.

Here’s a polished **README.md** version of your Docker learning journey and automation notes, formatted cleanly for GitHub:

```markdown
# 🐳 Docker Learning Journey & Automation Guide

---

### Day 01 🚀 Introduction to Docker
- 🐋 What is Docker?  
- 📜 History & Benefits  
- 🆚 Containerization vs Virtualization  
- ⚠️ Problems with traditional deployments  
- 🐳 Why the Docker whale logo  
- 🔗 Links to resources for learning and practicing  

---

### Day 02 🏗️ Docker Architecture
- 🧩 Components: Client, Daemon, Images, Containers & Registry  
- 🔄 Docker Workflow  
- 🖥️ Ran my first container and accessed it on `localhost:8080`  
- 🔗 Links to resources for learning and practicing  

---

### Day 03 💻 Develop with Containers
- 🤔 Why Develop with Containers?  
- 🆚 Traditional Development vs Develop with Containers  
- 🐳 Container-Based Development (Docker)  
- 🛠️ Hands-On: Develop with Containers  

---

### Day 04 🏗️ Docker Image Layers and Dockerfile Basics
- 🧱 Immutable Layers  
- ⚡ Union Filesystem  
- ✍️ Writable Layer  
- 📝 Dockerfile & Push to Docker Hub  

---

### Day 05 🏗️ Mastering Multi-Stage Docker Builds for React
- 🚀 Introduction  
- 🎯 Why Multi-Stage Builds?  
- 📚 Prerequisites  
- 🛠️ Step-by-Step Tutorial  
- 🔍 How It Works: Deconstructing the Dockerfile  
- ⚠️ Inspecting the Final Container  
- 🗑️ What Happens if You Delete `index.html`  
- ♻️ Restoring Deleted Files  
- 🧰 Useful Docker Commands  
- ✅ Best Practices  
- 🔗 Resources  

---

### Day 06 🌐 Docker Networking (Ports)
- 🛡️ Container isolation basics  
- 🔌 Publishing ports with `-p`  
- ⚡ Ephemeral ports  
- 📌 EXPOSE vs `-p` vs `-P`  
- 🔒 Security considerations  
- ➡️ Learn how traffic flows: browser → host → container  

---

### Day 07 ⚙️ Overriding Container Defaults in Docker
- 🐳 Images = defaults, containers = runtime  
- 🔁 Ports – Avoid conflicts with `-p HOST:CONTAINER`  
- 🌱 Env Vars – Pass configs at runtime / `.env`  
- ⚡ Resources – Limit CPU & memory  
- 🌐 Networking – Default vs custom networks  
- ▶️ CMD & ENTRYPOINT – Override container start  
- 🏁 Takeaway – Full control over ports, env, resources, network  

---

### Day 08 📦 Persisting Container Data in Docker
- ❌ Containers forget data  
- 📝 Layers Recap – Writable layer stores runtime changes  
- ⚠️ Problem – Data lost on container removal  
- ✅ Solution – Use Docker Volumes 📦  
- 🏗️ Create Volume – `docker volume create <name>`  
- 🔍 Inspect Volume – `docker volume ls / inspect`  
- ▶️ Run with Volume – `docker run -v <volume>:<path>`  
- 🏷️ Flags Explained – `-d`, `-p`, `-v`, `--name`  
- 🧠 Takeaway – Volumes = container memory 💾  

---

### Day 09 🐳 Docker Compose
- 🐙 What & Why – Multi-container apps, easier dev & deploy  
- ✨ Benefits – Simple control, collaboration, portability  
- 🖥️ Setup – Docker Desktop + YAML file  
- ⚡ Key Commands – `up`, `down`, `logs`, `ps`, `watch`  
- 🐍 Example App – Flask + Redis  
- 🔄 Compose Watch – Live code sync  
- 🧩 Modular Files – Split services for bigger apps  
- 🏁 Takeaway – Fast, repeatable, multi-container workflow ✨  

---

### Day 10 🐳 Docker Cheatsheet – Quick Reference

#### 🔹 Docker Basics
- 🖥️ `docker --version` → Check version  
- 📊 `docker info` → System info  
- 🔑 `docker login` / `docker logout` → Docker Hub access  

#### 🔹 Images
- 🗂️ `docker images` → List images  
- 📥 `docker pull <image>` → Download image  
- 🏗️ `docker build -t <name>:<tag> .` → Build image  
- 🗑️ `docker rmi <image>` → Remove image  
- 🏷️ `docker tag <image> <repo>:<tag>` → Tag image  

#### 🔹 Containers
- 📦 `docker ps` → Running containers  
- 📦 `docker ps -a` → All containers  
- ▶️ `docker run <image>` → Run container  
- 🔌 `docker run -d -p 8080:80 <image>` → Run detached + port mapping  
- ⏯️ `docker start/stop <container>` → Start/Stop  
- 🗑️ `docker rm <container>` → Remove container  

#### 🔹 Logs & Access
- 📜 `docker logs <container>` → View logs  
- 🖱️ `docker exec -it <container> /bin/bash` → Open shell  

#### 🔹 Cleanup
- 🧹 `docker system prune` → Remove unused resources  
- 🧹 `docker system prune -a` → Aggressive cleanup  

---

### Day 11 🐳 Automating Docker Builds with GitHub Actions

#### 📖 Introduction
Why automate Docker builds? → Manual build/tag/push is repetitive.  

#### ❌ Problem
- Slow, error-prone, repetitive manual steps.  

#### 🤖 Solution
- GitHub Actions automates build, tag, and push.  

#### 🛠️ Prerequisites
- Docker Hub account  
- Dockerfile ready  
- Credentials (`DOCKER_USERNAME`, `DOCKER_PASSWORD`)  

#### 🔑 Setup
- Add secrets in GitHub →  
  - `DOCKER_USERNAME` (your Docker Hub username, e.g., `970371`)  
  - `DOCKER_PASSWORD` (Docker Hub token).  

#### 📂 Workflow File
- Create `.github/workflows/docker-ci.yml`  

#### 🪜 Workflow Steps
1. 📥 Checkout code  
2. 🏷️ Extract metadata (tags, annotations)  
3. 🔑 Log in to Docker Hub  
4. ⚙️ Set up Docker Buildx  
5. 🏗️ Build & push image  

#### ✅ Result
- Every commit to `main` → Auto-build & push to Docker Hub.  

#### 🚀 Benefits
- Saves time  
- Reduces errors  
- Improves collaboration  

---

