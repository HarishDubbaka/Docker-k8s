# 🧑‍💻 Develop with Containers 

Yesterday, we successfully ran our **first Docker container** 🎉  
Today, we move one step further and begin **application development using containers**.

The goal of this learning is **not just to run commands**, but to clearly understand:
- **Why** we use them
- **What happens behind the scenes**
- Clarity always comes before complexity.
---

## 🎯 What You Will Learn Today

In this session, you will:

- Clone and start a containerized development project
- Work with both **backend** and **frontend** code
- See application changes **immediately**
- Understand how Docker enables **real-time development**

---

## 📦 Why Develop with Containers?

Traditional development setups often lead to:

- Different environments for different developers
- Dependency and version conflicts
- The classic *“It works on my machine”* problem

Docker solves this by:

- Packaging the application and dependencies together
- Providing a consistent environment everywhere
- Making setup fast, repeatable, and reliable

---

## 🏗️ Development Workflow (High Level)

The containerized development flow looks like this:

1. Clone the development project locally
2. Start the application using Docker
3. Source code is mounted into containers
4. When code changes are made:
   - No rebuild is required
   - Changes appear instantly in the browser

This creates a **fast and efficient feedback loop**.

---

## 🔄 Instant Feedback Loop

One of the biggest advantages of developing with containers:

- Edit code locally
- Containers automatically detect changes
- Refresh the browser to see updates instantly

This improves **productivity, confidence, and learning speed**.

## 🐳 Why the Container Still Shows Old Code

- Containers are immutable: once created, they run from the image snapshot.
- If you change your source code locally, the running container won’t see it unless you rebuild the image.
- Restarting the container (docker restart) only restarts the same image — it doesn’t rebuild with your new code.

---

## ✅ Prerequisites

Before starting, ensure:

- Docker Desktop is installed
- Docker Desktop is running
- Basic understanding of Docker containers

---

# ⚖️ Traditional Development vs Develop with Containers

This comparison explains **why container-based development is used in real projects**.

---

## 🖥️ Traditional Development

### How It Works
- Install runtimes (Node.js, Java, Python, etc.) manually
- Install dependencies on the host machine
- Environment differs from system to system

### Challenges
- Environment inconsistency
- Dependency conflicts
- Slow onboarding
- Hard to replicate Dev/Test/Prod

---

## 🐳 Container-Based Development (Docker)

### How It Works
- Application runs inside Docker containers
- Docker Compose manages services
- Same environment for everyone

### Benefits
- Consistent environments
- Faster onboarding (clone → run)
- No dependency conflicts
- Cleaner host system

---

## 📊 Comparison Table

| Feature | Traditional | Containers |
|------|------------|------------|
| Setup | Manual | Automated |
| Consistency | Low | High |
| Dependency Conflicts | Common | Eliminated |
| Onboarding | Slow | Fast |
| Portability | Limited | High |

---

## 🧑‍💻 Hands-On: Develop with Containers

In this exercise, we use a **Docker-provided To-Do application**.

---

## 🚀 Start the Project

### 📥 Clone the Repository

```bash
git clone https://github.com/docker/getting-started-todo-app
````

This downloads the application source code to your local system.

---

### 📂 Navigate to the Project Directory

```bash
cd getting-started-todo-app
```

---

## 🐳 Start the Development Environment

We use **Docker Compose** to start the application.

```bash
docker compose watch
```

### What this does:

* Pulls required container images
* Starts all required services
* Watches for source code changes during development

Docker handles the entire setup automatically.

---

## 🌐 Access the Application

Open your browser and visit:

```
http://localhost
```

⏳ The application may take a minute to start.

---

## ✅ Explore the App

You’ll see a simple **To-Do application**.

Try:

* Adding tasks
* Marking tasks as completed
* Deleting tasks

This confirms the application is running successfully inside containers.

---

## 🧩 What’s Running in the Environment?

At a high level, the development environment consists of multiple containers:

* **React Frontend**
  A Node.js container running the React dev server using Vite.

* **Node Backend**
  Provides APIs to create, retrieve, and delete to-do items.

* **MySQL Database**
  Stores the application data.

* **phpMyAdmin**
  Web-based database UI available at:

  ```
  http://db.localhost
  ```

* **Traefik Proxy**
  Routes traffic:

  * `/api/*` → Backend
  * `/` → Frontend
  * `db.localhost` → phpMyAdmin

All services are accessible via **port 80**, removing the need for multiple ports.

👉 As a developer, you **don’t need to install or configure anything manually**.
Docker Desktop handles everything.

---

## 🔍 Verifying Running Containers

You can verify all running services using:

```bash
docker ps
```

You can also view and manage containers directly from **Docker Desktop**.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/2207aa13221374e4e80279d95d31e1839a7aff4e/Day03/todo.png)

---

## 🎯 Key Takeaways

* Containers can be used for **full development environments**
* No manual dependency installation required
* Same behavior across all systems
* Faster, cleaner, and more reliable development

---

## 📌 What’s Next?

## 🚀 Container Learning Journey

Welcome to our containerization journey! This guide outlines the next steps to level up your skills.

1. **Package your application as a container image**  
   - Write a `Dockerfile` that defines how your app should be built.  
   - Use `docker build` to create the image locally.  

   ```bash
   docker build -t <your-username>/<image-name>:<tag> .

Stay curious and keep learning 🚀🐳

---

📘 **Reference Documentation**
[https://docs.docker.com/](https://docs.docker.com/)



