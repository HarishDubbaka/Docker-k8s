# 📦 Persisting Container Data in Docker

When a container starts, it uses the files and configuration provided by the image. Each container can create, modify, and delete files, and this does **not** affect other containers.

However, when the container is **deleted**, all those file changes are also **deleted** ❌.

While this **ephemeral nature of containers** is great, it poses a challenge when you want to **persist data**.
For example, if you restart a database container, you probably **don’t want to start with an empty database** 😬.

So… how do you persist files? 🤔

---

## 🔁 Recall: Day04 – Image & Container Layers

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

📝 **All file changes go into the writable layer**

### When the container is deleted:

* ❌ Writable layer is removed
* ✅ Image layers are reused

---

## 🧪 Let’s Do an Example

We’ll create files inside a Docker container and see what happens.

---

## 🚀 Get Started

### 1️⃣ Clone the Sample Repository

Clone a sample Git repository (or use your own project):

```bash
git clone https://github.com/docker/getting-started-app.git
```

### 2️⃣ Move Into the Directory

```bash
cd getting-started-app/
```

### 3️⃣ Create an Empty Dockerfile

```bash
touch Dockerfile
```

### 4️⃣ Add Content to Dockerfile

Using your preferred text editor, paste the following:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

📌 *(Details about each instruction were already shared in the video)*

---

## 🏗️ Build the Docker Image

```bash
docker build -t dockervolume-todo .
```

### ✅ Verify the Image

```bash
docker images
```

---

## ▶️ Run the Container

```bash
docker run -dp 3000:3000 dockervolume-todo
```

### 🔍 Check Running Containers

```bash
docker ps
docker ps -a
```

⚠️ Make sure no other service is using port `3000`
Otherwise you may see:

```
Bind for 0.0.0.0:3000 failed: port is already allocated
```

---

## 🐚 Access the Container

```bash
docker exec -it <container_id> sh
```

Inside the container, we created a directory and some files 📂.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/06d27857bc6a0bd050bcbbf51a8fa5b0e6176fdf/Day08/docker%20volume%20use%201st%20contiaer.png).

---

## ❗ Problem: Data Loss

Unfortunately, if the Docker container is **removed or deleted**, the data is lost 😕.

As we know:

* Docker containers are **ephemeral**
* Restarting a container ❌ does NOT delete data
* Deleting/removing a container ❌❌ DOES delete data

So how do we **persist data**? 🤔

---

## 🛑 Stopping & Deleting a Docker Container

### Step 1️⃣: Stop the Container

```bash
docker stop <container_name_or_id>
```

### Step 2️⃣: Remove the Container

```bash
docker rm <container_name_or_id>
```

### 🔁 One-Liner (Force Stop + Remove)

```bash
docker rm -f confident_jemison
```

### 🧼 Clean All Stopped Containers

```bash
docker container prune
```

⚠️ You’ll be asked for confirmation.

---

## 🔄 What Happens Next?

Now we run the **same Docker image again**, but:

* ❌ The data created earlier is **lost**
* ❌ The `harish` folder will **not exist** inside the container

To solve this, we need **Docker Volumes** 📦.

---

## 📊 Docker Volumes vs Storage Drivers

### 🗄️ Container Volumes

Volumes are a **storage mechanism** that allow data to persist **beyond the lifecycle of a container**.

Think of a volume like a **shortcut (symlink)** from inside the container to outside the container 🔗.

---

## 🧱 Create a Docker Volume

Example: Create a volume named `Harish-data`

```bash
docker volume create Harish-data
```
![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/004cd0f4a38ac939211ec432570201e3b3ff09ae/Day08/docker%20volume%20creates.png).

📍 Docker stores volumes inside Linux paths like:

```
/var/lib/docker
```

(Inside the Docker Linux VM)

---

## 🔍 Inspect Docker Volumes

You **don’t need** to `cd` into `/var/lib/docker`.

Use Docker CLI instead:

```bash
docker volume ls
docker volume inspect <volume_name>
```
![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/c61b368e9623db6175deedf19b5d95bdc7a0009c/Day08/docker%20inspect%20.png).
This shows:

* Where the volume is stored
* How it’s mounted

---

## ▶️ Run Container with Volume Mounted

```bash
docker run -dp 3000:3000 \
-v Harish-data:/app/data \
--name dockervolumepresistant-todo \
dockervolume-todo
```
![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/26362087bdb5ecd3dee81df4d8ea174a93943194/Day08/docker%20volume%20use%201st%20contiaer.png). 

![Image Alt](image_url).

---

## 🔎 What Each Part Means

| Part                                 | Purpose                         |
| ------------------------------------ | ------------------------------- |
| `docker run`                         | Start a new container           |
| `-d`                                 | Detached (background) mode      |
| `-p 3000:3000`                       | Map host port to container port |
| `-v Harish-data:/app/data`           | Mount volume to container       |
| `--name dockervolumepresistant-todo` | Container name                  |
| `dockervolume-todo`                  | Image name                      |

🧠 **This ensures data persistence even if the container is deleted.**

---

## 📸 Example Screenshot

👉 *(Add example screenshot here)*

---

If you want, I can:

* ✨ Add **more emojis**
* 🧹 Fix grammar without changing meaning
* 📘 Convert this into **Day05 Docker Notes**
* 🖼️ Add **diagram images**

Just say the word 😄
