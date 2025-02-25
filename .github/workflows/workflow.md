# Deploy the Conduit project with GitHub Actions CD Pipeline

This GitHub Actions workflow automates cloning and/or updating the Conduit project. Depending on the configuration, it is triggered manually or through the push process and ensures that the latest project version is always available.

## Table of Contents

1. [Workflow Process](#workflow-process)  
1. [Required secrets and variables](#required-secrets-and-variables)

## Workflow Process

The deployment pipeline is defined [here](./deployment.yml).

* There are several ways to **trigger the workflow**:
  * <ins>Manual Trigger</ins> with workflow_dispatch: The workflow can be started manually via the GitHub Actions tab.
  * <ins>Push to the main branch</ins>: You can set the workflow to be triggered automatically after a commit has been pushed to the main branch.

* The CD pipeline consists of **two jobs**:
  * 1.) <ins>**build job**</ins>: This job builds the Docker container images for the Conduit frontend and backend applications within the **GitHub Actions Runner** and pushes them to the **GitHub Container Registry (GHCR)**:
    * **Clone the repository** into the GitHub Actions Runner using [actions/checkout](https://github.com/actions/checkout).
    * Log in to the **GitHub Container Registry (GHCR)** using [actions/login-action](https://github.com/docker/login-action).
    * **Create .env-file** with required environment variables.
    * **Build** the backend and frontend **Docker images** in GitHub Actions Runner and pushes them to GHCR using [docker/build-push-action](https://github.com/docker/build-push-action)
    * **Store** the **deployment files** for the next job using [actions/upload-artifact](https://github.com/actions/upload-artifact)

  * 2.) <ins>**deploy job**</ins>: This job transfers the deployment files to the target server and starts the Conduit application using **Docker Compose**:
    * **Download** the **deployment files** into GitHubActions Runner using [actions/download-artifact](https://github.com/actions/download-artifact)
    * **Transfer** the deployment files via SCP **to target server** using [appleboy/scp-action](https://github.com/appleboy/scp-action)
    * **Start the containers** on the target server via SSH using [appleboy/ssh-action](https://github.com/appleboy/ssh-action) and the following Docker commands:
      * Stop running containers and clean up old resources:
        ```bash
        docker compose down
        docker rmi conduit-frontend conduit-backend
        docker volume rm conduit_conduit-volume
        docker builder prune -a -f
        ```
      * Start the frontend and backend containers in detached mode:
        ```bash
        docker compose up -d
        ```

* Logs are available in the GitHub Actions Runs. If an error occurs, the workflow stops and a notification is sent.

## Required secrets and variables

In order to run the workflow and create the .env for the cloning process, it requires various **GitHub Actions secrets and variables** that need to be set for the repository.  
  
<ins>You can define them under</ins>:
* Conduit repository > Settings > Security - Secrets and variables > Actions > Repository Secrets / Repository Variables

| Key | Description |
| --- | ----------- |
| **Secrets for workflow** | |
| SSH_HOST | IP address of remote server |
| SSH_USER | username on remote server |
| SSH_PRIVATE_KEY | Private SSH key of remote server |
| DEPLOY_DIR | Path to the folder in which the project should be published |
| **Secrets for backend application used in .env** | |
| SECRET_KEY | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| DJANGO_SUPERUSER_USERNAME | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| DJANGO_SUPERUSER_EMAIL | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| DJANGO_SUPERUSER_PASSWORD | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| **Variables for frontend and backend application used in .env** | |
| DEBUG | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| CORS_ORIGIN_WHITELIST | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| IP_ADDRESS_VM | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| BACKEND_EXTERNAL_PORT | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| FRONTEND_EXTERNAL_PORT | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
| API_URL | [further explanation](https://github.com/SarahZimmermann-Schmutzler/Conduit?tab=readme-ov-file#usage) |
