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

1. Make the `entrypoint.sh` **executable** in the conduit-backend directory if necessary:

    ```bash
    chmod 777 entrypoint.sh
    ```

1. Configure the **environment variables**:
    * Copy the content of the [`example.env`](./example.env) file into an .env file.

    * The new `.env` file should contain all the environment variables necessary to run the frontend and backend application!

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

1. It may be that the `entrypoint.sh` file of the conduit-backend needs to be made **executable**:
    * View rights of the files in the conduit-backend directory:

    ```bash
    ls -l
    ```

    * Set execute permissions if they don't exist:

    ```bash
    chmod 777 entrypoint.sh
    ```

1. Configure the **environment variables**:
    * Copy the content of the [`example.env`](./example.env) file into an .env file:

    ```bash
    cp example.env .env
    ```

    ```bash
    sudo nano .env
    ```

    * The new `.env` file should contain all the environment variables necessary to run the frontend and backend application:

    ```bash
    # variables for conduit-backend

    # essential cryptographic key used by Django to protect sensitive data and provide security-critical functionality
    SECRET_KEY=

    # set False for production mode 
    DEBUG=True

    # adds the host server (Frontend, IP:8282) to the CORS_ORIGIN_WHITELIST list
    IP_ADDRESS_VM_PORT=

    # adds the host server to the ALOWED_HOSTS list
    IP_ADDRESS_VM=

    # creates a superuser for the admin panel
    DJANGO_SUPERUSER_USERNAME=
    DJANGO_SUPERUSER_EMAIL=
    DJANGO_SUPERUSER_PASSWORD=


    # variables for conduit-frontend

    # adds backend server (IP:8383/api) to the environment.ts
    API_URL=
    ```

### Containerization with Docker Compose

#### The Files

1. The [`compose.yaml`](./compose.yaml) is responsible for managing and orchestrating the frontend and backend container. It defines what configurations they should have:

    ```bash
    version: "3.8"
    
    services:
        frontend:
            build:
                context: ./conduit-frontend
                args:
                    # passes the value from .env to Dockerfile for building process
                    - API_URL=${API_URL}
            env_file: .env
            container_name: conduit-frontend
            ports:
                - 8282:80
            restart: on-failure
            networks:
                - conduit-network

        backend:
            build:
                context: ./conduit-backend
            env_file: .env
            container_name: conduit-backend   
            volumes:
                # mount for the database directory
                - conduit-volume:/app/db
            ports:
                - 8383:8000
            restart: on-failure
            networks:
                - conduit-network

    volumes:
        conduit-volume:

    networks:
        conduit-network:
    ```

    * One service each is created for the frontend and the backend application:
        * Both:
            * take the build context (Dockerfile) from the corresponding folders
            * name their container (conduit-frontend and conduit-backend)
            * use the .env file to set variables
            * are in the same network (conduit-network)
            * restart in the event of an error that causes the container to terminate (restart on-failure)
            * have a port mapped through which they can be reached from the Internet (frontend:8282 and backend:8383)
    * **Frontend-Service**:
        * The args-parameter passes the value of `API_URL` (URL of backend to connect frontend and backend application) from .env to Dockerfile so that it is available during the build process.
    * **Backend-Service**:
        * It creates a volume (conduit-volume), i.e. a persistent data storage, for the sqlite database.

1. The [`Frontend-Dockerfile`](https://github.com/SarahZimmermann-Schmutzler/conduit-frontend/blob/master/Dockerfile) describes how a single Docker image for the frontend-container should be created:

     ```bash
    # 1: Build the Angular application
    # Base image for the build step
    FROM node:20-alpine AS build

    # Set the working directory inside the container
    WORKDIR /app

    # Copy only dependency files first
    COPY package.json package-lock.json $WORKDIR

    # Install dependencies with npm
    RUN npm install --legacy-peer-deps --prefer-offline --no-audit

    # Copy all project files into the container
    COPY . $WORKDIR

    # Define a build-time variable for the API URL
    ARG API_URL

    # Set an environment variable for runtime, initialized with the ARG value
    ENV API_URL=${API_URL}

    # Build Angular frontend
    RUN echo "export const environment = { production: true, apiUrl: '$API_URL' };" > src/environments/environment.ts && \
         echo "export const environment = { production: true, apiUrl: '$API_URL' };" > src/environments/environment.development.ts && \
         #cat src/environments/environment.ts && \
         #cat src/environments/environment.development.ts && \
         npm run build


    # 2: Serve the application using Nginx
    # Lightweight Nginx image for serving the application
    FROM nginx:1.26.2-alpine

    # Copy the built Angular application from the previous stage into the Nginx HTML directory
    COPY --from=build /app/dist/angular-conduit /usr/share/nginx/html

    # Expose port 80 for the Nginx server
    EXPOSE 80

    # Start Nginx
    # standard command in ngnix:1.26.2-alpine image is CMD ["nginx", "-g", "daemon off;"]
    ```

1. Also the [`Backend-Dockerfile`](https://github.com/SarahZimmermann-Schmutzler/conduit-backend/blob/master/Dockerfile) serves as the basis for the corresponding service in the Docker Compose file:

    ```bash
    # Stage 1: Build the dependencies in an isolated environment
    # base image (specific to the app's requirements)
    FROM python:3.6-slim AS builder

    WORKDIR /app

    # Copy only the requirements file to install dependencies
    COPY requirements.txt $WORKDIR

    # Upgrade pip and install dependencies
    RUN pip install --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

    # Stage 2: Copy application files into a lightweight image
    FROM python:3.6-slim

    WORKDIR /app

    # Copy only the installed dependencies from the builder stage
    COPY --from=builder /usr/local /usr/local

    # Copy the project files into the container
    COPY . $WORKDIR

    # Make the entrypoint script executable
    RUN chmod +x /app/entrypoint.sh

    # Expose the port for Gunicorn
    EXPOSE 8000

    # Use the entrypoint script to start the app
    ENTRYPOINT ["/app/entrypoint.sh"]
    ```

1. The [`entrypoint.sh`](https://github.com/SarahZimmermann-Schmutzler/conduit-backend/blob/master/entrypoint.sh) is used in combination with the backend's Dockerfile to initialize and configure the backend-container and executing the main command:

    ```bash
    #!/usr/bin/env bash

    # Exit immediately if any command exits with a non-zero status
    set -e


    # Step 1: Run database migrations
    echo "Running database migrations..."
    python manage.py makemigrations || { echo "Makemigrations failed"; exit 1; }
    python manage.py migrate || { echo "Migration failed"; exit 1; }


    # Step 2: Create a superuser
    echo "Creating superuser..."
    python manage.py createsuperuser --noinput \
        --email "$DJANGO_SUPERUSER_EMAIL" \
        --username "$DJANGO_SUPERUSER_USERNAME"


    # Step 3: Start the Django server 
    #echo "Starting the server..."
    python manage.py runserver 0.0.0.0:8000
    ```

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
