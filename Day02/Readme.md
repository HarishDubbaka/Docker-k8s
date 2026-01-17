Docker Architecture and Components

Docker Architecture

Docker architecture involves several key components, each playing a vital role in the containerization process:

Docker Client – Triggers interactions with the Docker Host.

Docker Host – Runs a Docker daemon and a Docker registry.

Docker Daemon – Runs inside the Docker Host and handles images and containers.

Docker Registry – Stores Docker images. Docker Hub is a public registry, but private registries can also be set up.

Docker Workflow

The workflow of Docker components communicating with each other:

A CLI (client) issues a build command to the Docker Host. The Docker daemon builds an image based on the client's inputs. This image is saved in the Docker registry (local or Docker Hub).

The Docker daemon either creates a new image or pulls an existing image from the Docker registry.

The daemon creates an instance of a Docker image.

The client issues a run command, triggering the creation of a container which then executes.

Docker Engine

Docker Engine is the core component of the Docker system. It is open-source and enables containerization.

Daemon Process (Server) – A background process responsible for managing images, containers, storage volumes, and networks.

Client (CLI) – Provides commands to interact with the daemon.

REST API – Used by the CLI and Docker daemon to communicate and execute instructions.

Docker Image

A Docker Image is the blueprint for containers.

Images contain pre-installed software, are portable, and can be reused.

They are created from Dockerfiles, which are text files containing instructions for building the image layer by layer.

Images serve as templates for Docker containers.

Docker Container

Containers are instances of Docker images.

Multiple containers can be created from a single image.

Containers package application code, dependencies, binaries, and configuration to run applications efficiently.

They are analogous to cargo containers: portable, manageable, and isolated.

Docker Registry

A repository used to store Docker images.

Public Registries – Docker Hub, similar to GitHub for code.

Private Registries – Organizations can host their own registries to store proprietary images securely.
