# Day 10 🚢 Docker Commands Recap – Quick Cheat Sheet

Sharing a quick recall of the Docker commands we’ve been using regularly while learning Docker. Super handy for beginners and daily practice 🚀

---

## 🔹 Docker Basics
- `docker --version` → Check Docker version  
- `docker info` → Docker system info  

## 🔹 Images
- `docker images` → List images  
- `docker pull <image>` → Download image  
- `docker build -t <name> .` → Build image  
- `docker rmi <image_id>` → Remove image  

## 🔹 Containers
- `docker ps` → Running containers  
- `docker ps -a` → All containers  
- `docker run <image>` → Run container  
- `docker run -d -p 8080:80 <image>` → Run in detached mode  
- `docker start <container>` / `docker stop <container>`  
- `docker rm <container>` → Remove container  

## 🔹 Logs & Access
- `docker logs <container>` → View container logs  
- `docker exec -it <container> /bin/bash` → Access container shell  

## 🔹 Cleanup
- `docker system prune` → Clean unused resources 🧹

---

📌 Still learning, not an expert — but **consistency beats perfection** 💪  
Hope this helps fellow Docker learners!


