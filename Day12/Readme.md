# 🚀 Day 12: Introduction to Azure Pipelines with Docker

This document explains how to use **Azure Pipelines** to **build and push Docker images to Docker Hub** automatically.  
It helps you create a **secure CI workflow** for containerized applications using Azure DevOps.

---

## 📌 Prerequisites

Before you begin, ensure you have the following:

- 🐳 A **Docker Hub account**
- 🔑 A **Docker Hub Access Token (PAT)**
- ☁️ An **active Azure DevOps project**
- 🔗 A Git repository connected to Azure DevOps
- 📄 A valid **Dockerfile** in the root directory (or correct build context)

---

## 🧭 Overview

This guide walks you through **building and pushing Docker images using Azure Pipelines**.

By the end of this setup, you will be able to:

- 🔐 Configure Docker authentication securely
- ⚙️ Set up an automated pipeline
- 📦 Build Docker images automatically
- 🚢 Push images to Docker Hub
- 🔄 Trigger builds on every push to the `main` branch

---

## 🔧 Step 1: Configure Docker Hub Service Connection (Optional but Recommended)

To securely authenticate Docker Hub with Azure Pipelines:

1. Go to **Azure DevOps → Project Settings**
2. Select **Service Connections**
3. Click **New Service Connection**
4. Choose **Docker Registry**
5. Select **Docker Hub**
6. Enter:
   - Docker Hub username
   - Docker Hub access token
7. Give it a name like:
```

my-docker-registry

```
8. Grant access **only to required pipelines**

> ⚠️ **Important:**  
> Avoid granting access to **all pipelines**.  
> Always follow the **principle of least privilege** for better security 🔒

---

## 🔑 Step 2: Create Docker Hub Access Token

1. Log in to **Docker Hub**
2. Navigate to:
```

Account Settings → Security → New Access Token

```
3. Give it a name (example: `AzureDevOpsPipeline`)
4. Copy the token immediately (you can’t view it again)

---

## 🔐 Step 3: Add Docker Token as Secret Variable in Azure DevOps

1. Go to your **Azure DevOps Pipeline**
2. Open **Variables**
3. Click **Add Variable**
4. Set:
- **Name:** `dockerHubPassword`
- **Value:** Docker Hub Access Token
- ✅ Enable **Keep this value secret**

> ❗ Never hardcode Docker passwords in YAML files.

---

## 👤 Step 4: Verify Docker Hub Username

Make sure your username is **exactly** your Docker Hub username:

✅ Correct:
```

yourusername

```

❌ Incorrect:
```

123456
[email@example.com](mailto:email@example.com)

````

---

## ⚙️ Azure Pipeline Configuration

The following pipeline will:

- Trigger on `main` branch
- Build a Docker image
- Tag it with:
  - Build ID
  - `latest`
- Push both tags to Docker Hub

---

## 📄 azure-pipelines.yml

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
````

---

## 🧠 Pipeline Explanation

### 🔁 Trigger

Runs automatically when code is pushed to the `main` branch.

---

### 🔐 Secure Login

Uses Docker access token stored as a **secret variable**, ensuring credentials are never exposed in logs.

---

### 🏗️ Build Image

Creates a Docker image from the Dockerfile in the repository.

---

### 🏷️ Tag Image

* `Build.BuildId` → Unique version per pipeline run
* `latest` → Always points to the most recent image

---

### 🚀 Push Image

Uploads the image to Docker Hub so it can be used for deployment.

---

### 🔓 Logout

Logs out from Docker Hub to clean up credentials and improve security.

---

## 🔒 Security Best Practices

* ✅ Use Docker Hub access tokens
* ✅ Store secrets in Azure DevOps variables
* ❌ Never commit secrets to Git
* ❌ Avoid giving global pipeline access

---

## ✅ Summary

✔ Automated Docker image builds
✔ Secure authentication
✔ CI-ready workflow
✔ Docker Hub integration
✔ Best practices followed

🎉 **Day 12 Completed Successfully!**

---

Happy Learning & Happy Shipping 🚢🐳

```

---

If you want, I can also:
- 📘 Simplify this for **beginners**
- 🧩 Add **interview questions**
- ☁️ Extend this to **AKS / Azure Web App**
- 📄 Convert it into **PDF or notes**

Just tell me 😊
```
