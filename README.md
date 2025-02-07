# Conduit

## Containerization of an Frontend and Backend application

This repository serves as a guide for containerizing a simple [Frontend](https://github.com/SarahZimmermann-Schmutzler/conduit-frontend) and associated [Backend](https://github.com/SarahZimmermann-Schmutzler/conduit-backend) application using **Docker Compose**.  
  
This Repository was created as part of my training at the **Developer Academy**.  

## Table of Contents

1. [Technologies](#technologies)  
1. [Description](#description)
1. [Quickstart](#quickstart)  
1. [Usage](#usage)  
    * [Installation and Preparation](#installation-and-preparation)
    * [Containerization with Docker Compose](#containerization-with-docker-compose)

## Technologies

* **Docker** 24.0.7
  * **Compose** v2.32.4 (module to install, [More Information](https://docs.docker.com/compose/))

## Description

### The Conduit-Project

**Conduit** was created to demonstrate a fully fledged application built with Angular that interacts with an actual backend server including CRUD operations, authentication, routing, pagination, and more.  
  
It is a complete full-stack blogging system that serves as a reference implementation for various frontend and backend technologies. It is often used for educational purposes to show developers how a real application is built using modern technologies.  

It is provided free by [Thinkster](https://thinkster.io/) and is also known as the **RealWorld Example App**.

> [!TIP]
> If you want to learn more about the [Frontend](https://github.com/SarahZimmermann-Schmutzler/conduit-frontend) and [Backend](https://github.com/SarahZimmermann-Schmutzler/conduit-backend) app or try them both individually, check out their repositories.

### Conduit combined in a container

Often the frontend and backend are hosted on different servers. A clear option is to host both together in a container. Here is a way to make it work.
  
## Quickstart

0) [Fork](https://docs.github.com/de/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) the project to your namespace, if you want to make changes or open a [Pull Request](https://docs.github.com/de/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).

1. [Clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) the project to your platform if you just want to use it:
    * <ins>Example</ins>: Clone the repo e.g. using an SSH-Key:  

    ```bash
    git clone git@github.com:SarahZimmermann-Schmutzler/Conduit.git
    ```

1. Init and update the **submodules**:

    ```bash
    git submodule init
    git submodule update
    ```

1. Configure the **environment variables**:
    * Copy the content of the [`example.env`](./example.env) file into an .env file.

    * The new `.env` file should contain all the environment variables - **all of them are required** to run the frontend and backend application!

1. **Build and start the containers** in the background (detached mode):

    ```bash
    docker compose up --build -d
    ```

1. Check whether the **containers are running** correctly:

* Browse to your **IP_ADDRESS:8282** to open the `Frontend`, you should see something like this:
    ![conduit-frontend](./frontend.png)

* Browse to your **IP_ADDRESS:8383/admin** to open the `Backend`, you should see this:
    ![conduit-backend](./backend.png)

## Usage

### Installation and Preparation

1. [Clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) the project to your platform if you just want to use it:
    * <ins>Example</ins>: Clone the repo e.g. using an SSH-Key:  

    ```bash
    git clone git@github.com:SarahZimmermann-Schmutzler/Conduit.git
    ```

1. Init and update the **submodules**:

    ```bash
    git submodule init
    git submodule update
    ```

1. Configure the **environment variables**:
    * Copy the content of the [`example.env`](./example.env) file into an .env file. It should contain all the environment variables - **all of them are required** to run the frontend and backend application:

    | Variable | Description | Type | Default Value |
    | -------- | ----------- | ---- | ------------- |
    | SECRET_KEY | Essential cryptographic key used by Django to protect sensitive data and provide security-critical functionality | string | 2^f+4@v7$v1f9yt0!s)2-1t$)tlp+&m17=*g))_xoi&&9m#2ax |
    | BACKEND_INTERNAL_PORT | Internal portnumber for backend-container | string | 8000 |
    | BACKEND_EXTERNAL_PORT | External portnumber for backend-container | string | 8383 |
    | FRONTEND_INTERNAL_PORT | Internal portnumber for frontend-container | string | 80 |
    | FRONTEND_EXTERNAL_PORT | External portnumber for frontend-container | string | 8282 |
    | DEBUG | Set False for production mode | boolean | True |
    | IP_ADDRESS_VM_WITH_PORT | Adds the host server (Frontend-IP with portnumber) to the CORS_ORIGIN_WHITELIST list | string | 127.0.0.1:8282 |
    | IP_ADDRESS_VM | Adds the host server to the ALLOWED_HOSTS list | string | 127.0.0.1 |
    | DJANGO_SUPERUSER_USERNAME | Username to create a superuser for the admin panel | string | admin |
    | DJANGO_SUPERUSER_EMAIL | Email address to create a superuser for the admin panel | string | admin@test.de |
    | DJANGO_SUPERUSER_PASSWORD | Password to create a superuser for the admin panel | string | changeMe123 |
    | API_URL | Adds backend server to the frontend (environment.ts) | string | http://127.0.0.1:8383/api |

### Containerization with Docker Compose

#### The Files

1. The [`compose.yaml`](./compose.yaml) is responsible for managing and orchestrating the frontend and backend container. It defines what configurations they should have:

    * One service each is created for the frontend and the backend application:
        * Both:
            * take the build context (Dockerfile) from the corresponding folders
            * name their container (conduit-frontend and conduit-backend)
            * use the .env file to set variables
            * are in the same network (conduit-network)
            * restart in the event of an error that causes the container to terminate (restart on-failure)
            * have a port mapped through which they can be reached from the Internet (frontend:8282 and backend:8383; can be customized via the .env)
    * **Frontend-Service**:
        * The args-parameter passes the value of `API_URL` (URL of backend to connect frontend and backend application) from .env to Dockerfile so that it is available during the build process.
    * **Backend-Service**:
        * It creates a volume (conduit-volume), i.e. a persistent data storage, for the sqlite database.

1. The [`Frontend-Dockerfile`](https://github.com/SarahZimmermann-Schmutzler/conduit-frontend/blob/master/Dockerfile) describes how a single Docker image for the frontend-container should be created:

    * This Dockerfile uses a multi-stage build to efficiently create and serve an Angular application with Nginx.
    * **Stage 1 (Build Stage)**:
        * Uses node:20-alpine for a lightweight Node.js environment.
        * Installs dependencies using npm install --legacy-peer-deps --prefer-offline for better compatibility and faster builds.
        * Copies the application files and builds the Angular project.
        * Dynamically sets the API_URL at build time and injects it into the Angular environment configuration.
    * **Stage 2 (Runtime Stage)**:
        * Uses nginx:1.26.2-alpine to serve the built Angular application.
        * Copies the compiled frontend from the build stage to the Nginx web root.
        * Exposes port 80 for serving the application.
        * Runs Nginx with the default startup command.

1. Also the [`Backend-Dockerfile`](https://github.com/SarahZimmermann-Schmutzler/conduit-backend/blob/master/Dockerfile) serves as the basis for the corresponding service in the Docker Compose file:

    * Uses a multi-stage build to create a lightweight and efficient Python 3.6-based container for this Django conduit-backend application
    * **Stage 1 (Builder Stage)**:
        * Uses python:3.6-slim as the base image.
        * Copies and installs dependencies from requirements.txt in an isolated environment to reduce the final image size.
    * **Stage 2 (Final Runtime Stage)**:
      * Copies only the necessary dependencies from the builder stage.
        * Copies the application source code into the container.
        * Exposes port 8000 for the application.
        * Sets the entrypoint script (entrypoint.sh) as executable and uses it to start the application.

1. The [`entrypoint.sh`](https://github.com/SarahZimmermann-Schmutzler/conduit-backend/blob/master/entrypoint.sh) is used in combination with the backend's Dockerfile to initialize and configure the backend-container and executing the main command:

    * The script exits immediately if any command fails, preventing further execution with errors.
    * **Step 1 - Database Migration**:
        * Runs makemigrations and migrate to apply any pending database changes - If migrations fail, the script exits with an error message.
    * **Step 2 - Superuser Creation**:
        * Creates a Django superuser using environment variables (DJANGO_SUPERUSER_EMAIL & DJANGO_SUPERUSER_USERNAME).
        * Runs in non-interactive mode (--noinput) to avoid prompts inside the container.
    * **Step 3 - Starting the Django Server**:
        * Launches the development server on 0.0.0.0:8000, making it accessible from outside the container.

#### The Use

1. **Build and start the containers** in the background (detached mode):

    ```bash
    docker compose up --build -d
    ```

    * To save the log files:

        ```bash
        docker logs <container-name> > backend-log.txt
        ```

        * Example for backend-container:

        ```bash
        docker logs conduit-backend > backend-log.txt
        ```

1. Check whether the **containers are running** correctly:

* Browse to your **IP_ADDRESS:8282** to open the `Frontend`, you should see something like this:
    ![conduit-frontend](./frontend.png)

* Browse to your **IP_ADDRESS:8383/admin** to open the `Backend`, you should see this:
    ![conduit-backend](./backend.png)

* If there is any **problem with the frontend-backend-connection** (f.e.CORS-Error) check if your ip-address is correctly set in the `.env`.

  * To check if the variables are correctly transferred into the container:

    ```bash
    docker compose exec backend python manage.py shell
    from django.conf import settings
    print(settings.CORS_ORIGINS_WHITELIST)
    print(settings.ALLOWED_HOSTS)
    ```

> [!NOTE]
> It is possible that not all features of the conduit app are working properly. It may be that paths for certain API endpoints in various frontend services need to be adjusted.
