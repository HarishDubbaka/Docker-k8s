# 🚀 Day 12: Introduction to Azure Pipelines with Docker 🐳☁️

This document explains how to use **Azure Pipelines** to **build and push Docker images to Docker Hub** automatically.
It helps you create a **secure CI workflow** for containerized applications using Azure DevOps.

> Just to inform you: while learning Azure Pipelines with Docker 🐳☁️, it’s completely normal to face errors ⚠️. Please be patient 🧘‍♂️ and take time to understand them 📘. Errors are part of the learning process, not a problem ✅. We’ll go step by step 🚶‍♂️🚀

---

## 📌 Prerequisites

Before you begin, ensure you have the following:

* 🐳 A **Docker Hub account**
* 🔑 A **Docker Hub Access Token (PAT)**
* ☁️ An **active Azure DevOps project**
* 🔗 A Git repository connected to Azure DevOps
* 📄 A valid **Dockerfile** in the root directory (or correct build context)

---

## 🧭 Overview

By the end of this setup, you will be able to:

* 🔐 Configure Docker authentication securely
* ⚙️ Set up an automated pipeline
* 📦 Build Docker images automatically
* 🚢 Push images to Docker Hub
* 🔄 Trigger builds on every push to the `main` branch

---

## 1️⃣ Create a New Project (if not already) 📂

1. Go to [Azure DevOps](https://dev.azure.com/)
2. Click **New Project** ➕
3. Fill in:

   * **Project Name** 📝
   * **Visibility** (Private/Public) 👀
4. Click **Create** ✅

---

## 2️⃣ Import Your Git Repository 🔗

1. Navigate to **Repos → Files**
2. Click **Import a repository** ⬇️
3. Paste your repository URL in the **Clone URL** field:

```text
https://github.com/yourusername/your-repo.git
```

4. Click **Import** 🏗️

> ⚠️ Ensure you have access permissions to the repository.

---

## 3️⃣ Verify Repository Import ✅

* Once the import completes, you should see all your **files and folders** in Azure DevOps Repos 📂
* Check that your **Dockerfile** and project files are present ✅

---

## 4️⃣ Configure Docker Hub Service Connection 🔧

1. Go to **Azure DevOps → Project Settings → Service Connections**
2. Click **New Service Connection → Docker Registry → Docker Hub** 🐳
3. Enter your **Docker Hub username** and **Access Token** 🔑
4. Give it a name, e.g.:

```text
my-docker-registry
```

5. Grant access **only to required pipelines** 🔒

> ⚠️ **Important:** Avoid granting access to **all pipelines**. Always follow the **principle of least privilege** 🔐

---

## 5️⃣ Create Docker Hub Access Token 🔑

1. Log in to **Docker Hub**
2. Navigate to:

```text
Account Settings → Security → New Access Token
```

3. Give it a name (e.g., `AzureDevOpsPipeline`) 🏷️
4. Copy the token immediately (you won’t be able to see it again) 📋

---

## 6️⃣ Add Docker Token as Secret Variable in Azure DevOps 🔐

1. Go to your **Azure DevOps Pipeline → Variables**
2. Click **Add Variable** ➕
3. Set:

   * **Name:** `dockerHubPassword` 🔑
   * **Value:** Docker Hub Access Token
   * ✅ Enable **Keep this value secret** 👀

> ❗ Never hardcode Docker passwords in YAML files 🚫

---

## 7️⃣ Verify Docker Hub Username 👤

Make sure your username is **exactly** your Docker Hub username:

✅ Correct:

```text
yourusername
```

❌ Incorrect:

```text
123456
email@example.com
```

---

## 8️⃣ Azure Pipeline Configuration ⚙️

The pipeline will:

* Trigger on the `main` branch 🔄
* Build a Docker image 🏗️
* Tag it with:

  * Build ID 🏷️
  * `latest` 🔖
* Push both tags to Docker Hub 🚢

---

## 📄 azure-pipelines.yml 📜

```yaml
trigger:
  - main

pr:
  - main

variables:
  dockerUsername: '970371'                # Docker Hub username
  dockerPassword: '$(dockerHubPassword)'  # Secret variable
  imageName: '970371/my-image'            # Docker Hub repository
  buildTag: '$(Build.BuildId)'
  latestTag: 'latest'

pool:
  vmImage: 'ubuntu-latest'                # Hosted agent with Docker installed

stages:
  - stage: BuildAndPush
    displayName: Build and Push Docker Image
    jobs:
      - job: DockerJob
        displayName: Build and Push
        steps:
          - checkout: self
            displayName: Checkout Code

          - script: |
              echo "$DOCKER_HUB_PASSWORD" | docker login -u $(dockerUsername) --password-stdin
            displayName: Docker Login
            env:
              DOCKER_HUB_PASSWORD: $(dockerPassword)

          - script: |
              docker build -t $(imageName):$(buildTag) .
              docker tag $(imageName):$(buildTag) $(imageName):$(latestTag)
              docker push $(imageName):$(buildTag)
              docker push $(imageName):$(latestTag)
            displayName: Build and Push Docker Image

          - script: docker logout
            displayName: Docker Logout
```

---

## 🧠 Pipeline Explanation

### 🔁 Trigger

Runs automatically when code is pushed to the `main` branch

### 🔐 Secure Login

Uses Docker access token stored as a **secret variable**, ensuring credentials are never exposed in logs

### 🏗️ Build Image

Creates a Docker image from the Dockerfile in the repository

### 🏷️ Tag Image

* `Build.BuildId` → Unique version per pipeline run
* `latest` → Always points to the most recent image

### 🚀 Push Image

Uploads the image to Docker Hub for deployment

### 🔓 Logout

Logs out from Docker Hub to clean up credentials and improve security

---

## 🔒 Security Best Practices 🛡️

* ✅ Use Docker Hub access tokens
* ✅ Store secrets in Azure DevOps variables
* ❌ Never commit secrets to Git
* ❌ Avoid giving global pipeline access

---

## 9️⃣ Verify Image in Docker Hub After Successful Push 🐳

Once the pipeline runs successfully, your Docker image is pushed to **Docker Hub** automatically.

### 🔍 Verify in Docker Hub

1. Log in to Docker Hub
2. Navigate to your repository 🗂️

![Image Alt](https://github.com/HarishDubbaka/Docker-k8s/blob/5677f0f69eaf36b59e91c9fb5ac0a3395b113097/Day12/azurepipeline.png)

No worries 😊

Just to inform you:

🐳 `970371/my-image` → built using **Azure DevOps Pipeline**

🐳 `970371/my-image1` → built using **GitHub Actions**

---

## ✅ Summary 🎉

* 🏗️ Automated Docker image builds
* 🔐 Secure authentication
* ⚙️ CI-ready workflow
* 🚢 Docker Hub integration
* ✅ Best practices followed

🎉 **Day 12 Completed Successfully!**

---

Happy Learning & Happy Shipping 🚢🐳

---
