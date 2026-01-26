When we create a Dockerfile, the usual workflow involves building the Docker image, testing it, and then pushing it to a Docker registry such as Docker Hub for storage and reuse.

> 💡 **Explanation:** A Dockerfile is a blueprint for your Docker image. After creating it, the typical workflow is:
>
> 1. Build the Docker image (`docker build`)
> 2. Test it locally (`docker run`)
> 3. Push it to Docker Hub (`docker push`) so it can be reused or deployed elsewhere

However, if we later need to update or fix something in the Dockerfile, the process becomes repetitive. Each change requires us to manually rebuild the image, add or update image tags, and push the new version back to Docker Hub

> 💡 **Explanation:** Every time you change the Dockerfile, you repeat these steps manually, which can be cumbersome, especially for larger projects or teams.

Over time, this manual approach becomes time-consuming, error-prone, and difficult to scale—especially when changes are frequent or multiple team members are involved.

❗ The Problem

Now imagine you need to fix or update something in the Dockerfile tomorrow:

🔁 You must rebuild the image

🏷️ Manually add or update the image tag

⬆️ Push it again to Docker Hub

This has to be done every single time there’s a change.

😵‍💫 Issues with this approach:

* Time-consuming
* Repetitive manual steps
* Easy to forget tags or push the wrong version
* Not scalable for teams or frequent updates

> 💡 **Explanation:** Manually performing these steps introduces human error and slows down development, especially for continuous updates.

To address this challenge, we can use GitHub Actions with Docker. GitHub Actions allows us to automate the entire workflow. Whenever changes are pushed to the repository, it can automatically build the Docker image, apply the correct tags, and push the updated image to Docker Hub without manual intervention.

> 💡 **Explanation:** GitHub Actions is a **CI/CD tool** built into GitHub. You can configure it to automatically execute workflows like building and pushing Docker images whenever certain events happen (e.g., code push).

By adopting this approach, we eliminate repetitive manual steps and ensure that our Docker images are always up to date, consistently tagged, and reliably published.

In short, GitHub Actions helps us automate Docker image builds

---

## Introduction to GitHub Actions with Docker

This guide provides an introduction to building CI pipelines using Docker and GitHub Actions. You will learn how to use Docker's official GitHub Actions to build your application as a Docker image and push it to Docker Hub. By the end of the guide, you'll have a simple, functional GitHub Actions configuration for Docker builds. Use it as-is, or extend it further to fit your needs.

---

### Prerequisites

If you want to follow along with the guide, ensure you have the following:

* A verified Docker account
* Familiarity with Dockerfiles

> 💡 **Explanation:** You need access to Docker Hub to push images and some understanding of Dockerfile syntax and image concepts.

This guide assumes basic knowledge of Docker concepts but provides explanations for using Docker in GitHub Actions workflows.

---

### Get the sample app

This guide is project-agnostic and assumes you have an application with a Dockerfile.

If you need a sample project to follow along, you can use this sample application, which includes a Dockerfile for building a containerized version of the app. Alternatively, use your own GitHub project or create a new repository from the template.

```bash
git clone https://github.com/dvdksn/rpg-name-generator.git
```

> 💡 **Explanation:** This sample app is a ready-made project to practice building and pushing Docker images using GitHub Actions.

---

### Configure your GitHub repository

The workflow in this guide pushes the image you build to Docker Hub. To do that, you must authenticate with your Docker credentials (username and access token) as part of the GitHub Actions workflow.

For instructions on how to create a Docker access token, see **Create and manage access tokens**.

Once you have your Docker credentials ready, add the credentials to your GitHub repository so you can use them in GitHub Actions:

Make sure to have **full permissions** on Docker Hub

Open your repository's Settings.
Under **Security**, go to **Secrets and variables → Actions**.

* Under **Secrets**, create a new repository secret named `DOCKER_PASSWORD`, containing your Docker access token.
* Next, under **Variables**, create a `DOCKER_USERNAME` repository variable containing your Docker Hub username.

```bash
docker login -u 970371
```

> 💡 **Explanation:** GitHub Actions uses secrets and variables to securely store credentials. This ensures your Docker password is not exposed in code.

---

You don’t get this file automatically. You create it yourself inside your GitHub repository.

---

### ✅ Method 1: Create the Workflow File Locally (Recommended)

🔹 **Step 1:** Go to your project root

```bash
cd your-project-repo
```

🔹 **Step 2:** Create the required folders

```bash
mkdir -p .github/workflows
```

📌 This creates:

```
.github/
└── workflows/
```

🔹 **Step 3:** Create the workflow file

```bash
touch .github/workflows/docker-ci.yml
```

Now you have:

```
.github/workflows/docker-ci.yml
```

🔹 **Step 4:** Open and edit the file

```bash
nano .github/workflows/docker-ci.yml
```

(or use VS Code)

> 💡 **Explanation:** GitHub Actions workflows must be stored under `.github/workflows/` with a `.yml` or `.yaml` extension. Any workflow inside this folder is automatically detected.

---

### ✅ Method 2: Create Workflow File from GitHub UI (No Local Setup)

1️⃣ Go to your GitHub repository
2️⃣ Click **Actions** tab
3️⃣ Click **New workflow**
4️⃣ Choose **Set up a workflow yourself**
5️⃣ Name the file:

```
.github/workflows/docker-ci.yml
```

6️⃣ Paste your workflow YAML
7️⃣ Click **Commit changes**

🎉 Done!

> 💡 **Explanation:** Using the GitHub UI is easier for beginners who don’t want to set up files locally.

---

### 🧠 Important Rules to Remember

* The folder must be exactly: `.github/workflows/`
* File extension must be: `.yml` or `.yaml`
* Any file inside this folder is automatically detected by GitHub Actions
* Workflow runs only after pushing to GitHub

---

### Workflow YAML

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
          provenance: true
          sbom: true
```

> 💡 **Explanation of workflow steps:**
>
> 1. **Checkout** – pulls the repository code into the runner.
> 2. **Extract Docker image metadata** – automatically generates tags based on your repo, branch, and commit.
> 3. **Log in to Docker Hub** – authenticates using your secrets.
> 4. **Set up Docker Buildx** – enables advanced Docker build features.
> 5. **Build and push Docker image** – builds the Docker image and pushes it to Docker Hub, including metadata, SBOM (software bill of materials), and provenance information.

---

