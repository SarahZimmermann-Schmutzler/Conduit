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

* The CD pipeline runs in **several steps**:
  * Establish an **SSH connection** to the specified remote server.
  * **Navigation** to the intended project directory.
  * **Clone** repository, including  initializing submodules, if the Conduit project does nor exist yet.
  * **Update** the existing repository:
    * Stop and remove running containers and associated units.
    * Pull the newest version of the repository.
  * **Create** an **.env file** if it is necessary.
  * **Build and start the containers** for frontend and backend in detached mode with docker compose.

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
