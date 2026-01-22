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


📌 If you run:

```bash
docker run postgres
```

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

```bash
docker run -d -p 5432:5432 postgres   # First DB
docker run -d -p 5433:5432 postgres   # Second DB
```

### What’s happening?

* Inside both containers → Postgres listens on **5432**
* On your machine:

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

* All containers join it automatically
* Containers:

  * ❌ Can’t resolve each other by name
  * ⚠️ Less isolation

---

### ✅ Custom Network (Best Practice)

Create your own network:

```bash
docker network create mynetwork
```

Run container on it:

```bash
docker run -d \
  --network mynetwork \
  -e POSTGRES_PASSWORD=secret \
  -p 5434:5432 \
  postgres
```

### Why this is better:

* Containers can talk using **names**
* Better isolation 🔐
* Cleaner architecture

🧠 Example:

```text
app → postgres
```

(No IP addresses needed!)

---

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
