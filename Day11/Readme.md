# Automating Docker Builds with GitHub Actions – Plain Language Guide

---

## 1️⃣ The Problem

When you make a Dockerfile (a file that tells Docker how to build your app), the usual steps are:

1. Build the Docker image (`docker build`)
2. Test it (`docker run`)
3. Push it to Docker Hub (`docker push`)

> **Problem:** Every time you make a small change in the Dockerfile, you have to do all of this again manually.

Imagine tomorrow you need to fix something:

* 🔁 Build the image again
* 🏷️ Add or update the version tag manually
* ⬆️ Push it to Docker Hub

Doing this every time is:

* Slow
* Boring and repetitive
* Easy to make mistakes (wrong tags, forgot to push)
* Hard for teams to keep track

---

## 2️⃣ The Solution: GitHub Actions

Instead of doing it by hand, we can **let GitHub Actions do it automatically**.

**GitHub Actions can:**

* Watch your repository for changes
* Build the Docker image automatically
* Tag the image correctly
* Push it to Docker Hub

> **Think of it like a robot**: you make a change, push your code, and the robot does all the Docker work for you.

---

## 3️⃣ What You Need Before Starting

1. A **Docker Hub account**
2. A **Dockerfile** (your app must be containerized)
3. Basic knowledge of Docker concepts

> Optional: Use a sample project:

```bash
git clone https://github.com/dvdksn/rpg-name-generator.git
```

---

## 4️⃣ Add Your Docker Hub Credentials to GitHub

We need GitHub to log in to Docker Hub automatically.

1. **Create a Docker access token** (like a password, but safer)
2. Go to your GitHub repository → **Settings → Security → Secrets and variables → Actions**

Add:

* **Secret**: `DOCKER_PASSWORD` → Your Docker token
* **Variable**: `DOCKER_USERNAME` → Your Docker Hub username

> **Tip:** Never put your password in plain text; GitHub Secrets keeps it safe.

---

## 5️⃣ Create the Workflow File

GitHub Actions needs a special file called a **workflow** to know what to do.

### Method 1: On Your Computer

```bash
cd your-project
mkdir -p .github/workflows
touch .github/workflows/docker-ci.yml
```

Open it in an editor (VS Code or nano) and paste the workflow YAML.

### Method 2: On GitHub

1. Go to **Actions → New workflow → Set up a workflow yourself**
2. Name it `.github/workflows/docker-ci.yml`
3. Paste the workflow YAML
4. Commit the file

> **Key:** GitHub will automatically detect any `.yml` file in `.github/workflows/`.

---

## 6️⃣ Workflow YAML Explained

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Extract Docker image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ vars.DOCKER_USERNAME }}/my-image

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          annotations: ${{ steps.meta.outputs.annotations }}
          provenance: trueyt 
          sbom: true
```

---

### Step-by-Step in Simple Words

1. **Checkout** – GitHub copies your code so it can work with it.
2. **Extract Docker image metadata** – GitHub automatically decides the image name and version tag for you.
3. **Log in to Docker Hub** – Uses your secrets to sign in safely.
4. **Set up Docker Buildx** – Prepares Docker to build images efficiently.
5. **Build and push Docker image** – Actually builds the image and uploads it to Docker Hub.

> ✅ Result: Every time you push to `main`, GitHub automatically builds and uploads your Docker image. No manual work required.

---

## 7️⃣ Why This is Awesome

* Saves time (no manual builds)
* Reduces errors (tags are correct, images always pushed)
* Works automatically for teams
* Scales easily for multiple updates

> **Imagine:** You push a code change → GitHub Actions builds the Docker image → pushes it to Docker Hub → you get a new working image automatically.

---
I changed the image name in my workflow, and GitHub Actions automatically built and pushed the updated image. I confirmed the new image is available in Docker Hub, so everything is working smoothly.

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/45403930484208608e2b9171e9d39e69348872ba/Day11/dockerhub1.png)
