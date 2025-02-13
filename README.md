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
    | **Used in backend application** | | | |
    | SECRET_KEY | Essential cryptographic key used by Django to protect sensitive data and provide security-critical functionality | string | secret |
    | DEBUG | Set False for production mode | boolean | True |
    | IP_ADDRESS_VM | Server IP address, added to the ALLOWED_HOSTS list in settings.py | string | 127.0.0.1 |
    | CORS_ORIGIN_WHITELIST | List of all hosts with their portnumbers, added to settings.py | string | "127.0.0.1:8282,localhost:8282" |
    | DJANGO_SUPERUSER_USERNAME | Username to create a superuser for the admin panel | string | admin |
    | DJANGO_SUPERUSER_EMAIL | Email address to create a superuser for the admin panel | string | admin@test.de |
    | DJANGO_SUPERUSER_PASSWORD | Password to create a superuser for the admin panel | string | changeMe123 |
    | **Used in frontend application** | | | |
    | API_URL | Adds backend URL to environment.prod.ts, used in app/core/interceptors/api.interceptor.ts | string | http://127.0.0.1:8383/api |
    | **Used for containerization** | | | |
    | BACKEND_EXTERNAL_PORT | External portnumber for backend-container | string | 8383 |
    | FRONTEND_EXTERNAL_PORT | External portnumber for frontend-container | string | 8282 |

### Containerization with Docker Compose

#### The Files

1. One service each is created for the frontend and the backend application. The [`compose.yaml`](./compose.yaml) is responsible for managing and orchestrating the frontend and backend container. It defines what configurations they should have.

1. The [`Frontend-Dockerfile`](https://github.com/SarahZimmermann-Schmutzler/conduit-frontend/blob/master/Dockerfile) describes how a single Docker image for the frontend-container should be created.

1. Also the [`Backend-Dockerfile`](https://github.com/SarahZimmermann-Schmutzler/conduit-backend/blob/master/Dockerfile) serves as the basis for the corresponding service in the Docker Compose file.

1. The [`entrypoint.sh`](https://github.com/SarahZimmermann-Schmutzler/conduit-backend/blob/master/entrypoint.sh) is used in combination with the backend's Dockerfile to initialize and configure the backend-container and executing the main command.

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
