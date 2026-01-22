# ⚙️ Overriding Container Defaults in Docker (Crystal Clear)

Think of a **Docker image** as a **preconfigured machine** 🖥️
When you start a container, Docker follows the image’s **default instructions**.

👉 But Docker lets you **override those defaults at runtime** without changing the image.

---

## 🧩 1. How Containers Normally Start

Every Docker image defines:

* **What program starts** (`CMD` / `ENTRYPOINT`)
* **Which ports the app listens on**
* **Default environment settings**

Docker File
```bash
FROM debian:bullseye-slim

ENV PGDATA=/var/lib/postgresql/data
EXPOSE 5432

USER postgres
WORKDIR /var/lib/postgresql

CMD ["postgres"]
```

📌 If you run:

```bash
docker run postgres
```

✅ Starts program → postgres

✅ Listens on port → 5432

✅ Default env → PGDATA for database storage

Docker says:

> “Okay, I’ll start Postgres exactly the way the image author decided.”

But in real life, you often need **custom behavior**.

That’s where overrides come in 👇

---

## 🔁 2. Overriding Network Ports (Very Common)

### ❓ The Problem

* You want to run **two containers**
* Both listen on the **same internal port**
* Host ports **cannot** be reused ❌

### ✅ The Solution: Port Mapping

```bash
docker run -d -p HOST_PORT:CONTAINER_PORT image
```

### 🧠 Example (Postgres)

## 🔹 Dockerfile (same for both DBs)
```dockerfile
FROM postgres:17

# Default environment variables (can be overridden at runtime)
ENV POSTGRES_USER=admin \
    POSTGRES_PASSWORD=secret \
    POSTGRES_DB=mydb

EXPOSE 5432
CMD ["postgres"]
```

👉 This image always runs PostgreSQL on **port 5432 inside the container**.  

---

## 🔹 Running Two Independent Databases

### First Database (on host port 5432)
```bash
docker run -d \
  -p 5432:5432 \
  --name db1 \
  -e POSTGRES_USER=user1 \
  -e POSTGRES_PASSWORD=pass1 \
  -e POSTGRES_DB=db_one \
  my-postgres
```

- Container name → `db1`  
- Host port → `5432`  
- Internal DB → `db_one` with user `user1`  

---

### Second Database (on host port 5433)
```bash
docker run -d \
  -p 5433:5432 \
  --name db2 \
  -e POSTGRES_USER=user2 \
  -e POSTGRES_PASSWORD=pass2 \
  -e POSTGRES_DB=db_two \
  my-postgres
```

- Container name → `db2`  
- Host port → `5433`  
- Internal DB → `db_two` with user `user2`  

---

## 🔹 How to Connect
- First DB → `psql -h localhost -p 5432 -U user1 db_one`  
- Second DB → `psql -h localhost -p 5433 -U user2 db_two`  

---

### What’s happening?

✅ Both containers run the **same image** Postgres listens on **5432**, but each has its own **port mapping, user, password, and database**.  
✅ They are **completely isolated** — two separate PostgreSQL servers on your machine.  

  * First DB → `localhost:5432`
  * Second DB → `localhost:5433`

🎯 Result: No conflicts, full control

---

## 🌱 3. Environment Variables (How Apps Get Config)

Containers don’t ask questions.
They **read environment variables** to know how to behave.

### Example

```bash
docker run -e foo=bar postgres env
```

Inside the container:

```
foo=bar
```

🧠 This is how you pass:

* Passwords 🔐
* Modes (dev/prod)
* API keys
* Feature flags

---

### 🧼 Cleaner Way: `.env` File

Instead of long commands:

```env
POSTGRES_PASSWORD=secret
POSTGRES_DB=mydb
```

Run:

```bash
docker run --env-file .env postgres
```

✅ Cleaner
✅ Safer
✅ Production-friendly

---

## 🧠 4. Limiting CPU & Memory (Very Important)

### Default Behavior 🚨

Containers can use **ALL** your CPU and memory.

That’s dangerous.

---

### ✅ Set Limits

```bash
docker run \
  -e POSTGRES_PASSWORD=secret \
  --memory="512m" \
  --cpus="0.5" \
  postgres
```

### Meaning:

* 🧠 Max memory → **512 MB**
* ⚡ Max CPU → **half a core**

Monitor usage live:

```bash
docker stats
```

📊 This protects your system and other containers.

---

## 🌐 5. Docker Networking (Simple Mental Model)

### Default Bridge Network

- When you install Docker, it creates a default bridge network.
- Every container you run automatically joins this network unless you specify otherwise.
- Problems with default bridge:
- ❌ Containers cannot resolve each other by name (you’d need IP addresses).
- ⚠️ Less isolation — all containers are lumped together.
  
Think of it like a shared public WiFi: everyone is connected, but you need IPs to talk, and it’s messy.

---

## ✅ Create a Custom Network

```bash
docker network create mynetwork
docker network ls   # verify it exists
```

- `mynetwork` is a private Docker network.
- Only containers attached to it can communicate with each other.

---

## 🚀 Run PostgreSQL on the Custom Network

```bash
docker run -d \
  --network mynetwork \
  --name mydb \
  -e POSTGRES_PASSWORD=secret \
  -p 5434:5432 \
  postgres
```

### Explanation:
- `--network mynetwork` → attach container to the custom network.  
- `--name mydb` → container name (used for DNS resolution).  
- `-e POSTGRES_PASSWORD=secret` → sets password for the `postgres` user.  
- `-p 5434:5432` → map host port **5434** → container port **5432**.  
- `postgres` → official PostgreSQL image.  

---

## 🔗 Connect to the Database

### Option 1: From Host (requires `psql` installed)
```bash
psql -h localhost -p 5434 -U postgres
```

### Option 2: From Inside the Container
```bash
docker exec -it mydb psql -U postgres
```
---

## 🔒 Why Custom Networks Are Better
- Containers can talk using **names** (e.g., `app → mydb`).  
- Provides **better isolation** — only containers on the same network can see each other.  
- Results in a **cleaner architecture** for multi‑service apps.

## ▶️ 6. Overriding CMD and ENTRYPOINT

Images define **how they start**.
You can override that if needed.

---

### With `docker run`

```bash
docker run postgres docker-entrypoint.sh -h localhost -p 5432
```

Here:

* You’re telling Docker:

  > “Ignore the default command—use this instead.”

---

### With Docker Compose (Cleaner)

```yaml
services:
  postgres:
    image: postgres:18
    environment:
      POSTGRES_PASSWORD: secret
    command: ["-p", "5432"]
```

Run:

```bash
docker compose up -d
```

📌 Compose is better for repeatable setups.

---

## 🔐 7. Default Network vs Custom Network (One Look)

| Feature         | Default Bridge | Custom Network |
| --------------- | -------------- | -------------- |
| Auto-created    | ✅              | ❌              |
| Name resolution | ❌              | ✅              |
| Isolation       | Low ⚠️         | High 🔐        |
| Best for prod   | ❌              | ✅              |

---

## ✅ Final Takeaway (Remember This)

🧠 **Images = defaults**
🛠️ **docker run / compose = overrides**

You control:

* 🔁 **Ports** → avoid conflicts
* 🌱 **Env vars** → configure apps
* 🧠 **Resources** → protect system
* 🌐 **Networks** → isolate & connect
* ▶️ **CMD / ENTRYPOINT** → startup logic

Once you master this, Docker stops feeling “magical” and starts feeling **predictable and powerful** 💪🐳

---

👉 **Next step (natural progression):**
Understanding **Volumes vs Bind Mounts** — how containers **keep data alive** 💾

Say the word and I’ll break that down just as clearly.
