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

1. Make the `entrypint.sh` **executable** in the conduit-backend directory if necessary:

    ```bash
    chmod 777 entrypoint.sh
    ```

1. **Build and start the container** in the background (detached mode):

    ```bash
    docker compose up --build -d
    ```

## Usage

### Installation and Preparation

1. [Clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) the project to your platform if you just want to use it:
    * <ins>Example</ins>: Clone the repo e.g. using an SSH-Key:  

    ```bash
    git clone git@github.com:SarahZimmermann-Schmutzler/Conduit.git
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

    * The new `.env file` should contain all the environment variables necessary to run the frontend and backend application:

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

1. The [`Frontend-Dockerfile`](./conduit-frontend/Dockerfile) describes how a single Docker image for the frontend-container should be created:

     ```bash
    
    ```

1. Also the [`Backend-Dockerfile`](./conduit-backend/Dockerfile) serves as the basis for the corresponding service in the Docker Compose file:

1. The Entrypoint

#### The Use

1. **Build and start the container** in the background (detached mode):

    ```bash
    docker compose up --build -d
    ```

    * To view the log files:

        ```bash
        docker compose logs -f
        ```

    * To stop the container:

        ```bash
        docker compose stop <container-name>
        ```

    * To delete the container:

        ```bash
        docker compose down <container-name>
        ```

    * To list all containers that are operatet by Docker Compose:

        ```bash
        docker compose ps
        ```

1. Check whether the **server is running** correctly:

* If you have a *Minecraft account*: You can connect to the server on your cloud VM from your [Java Minecraft client](https://www.minecraft.net/de-de/download) on your computer and play Minecraft.

* Use a [website](https://mcstatus.io/) that offers status checks for Minecraft Servers or try to establish a connection to the Minecraft Server using the [python `mcstatus` module](https://github.com/py-mine/mcstatus).

  * <ins>mcstatus.io</ins>:
    ![mcsrv](./mcsrv.png)

  * <ins>python module `mctatus`</ins>:
    * Intall the module:

        ```bash
        python -m pip install mcstatus
        ```

    * Write a python script. A proper template is given [here](https://github.com/py-mine/mcstatus?tab=readme-ov-file#java-edition):

        ```bash
        from mcstatus import JavaServer
        # status for Java Edition Server

        # You can pass the same address you'd enter into the address field in minecraft into the 'lookup' function
        # If you know the host and port, you may skip this and use JavaServer("example.org", 1234)
        # server = JavaServer.lookup("example.org:1234")
        server = JavaServer("IP_ADDRESS_VM", 8888)

        # 'status' is supported by all Minecraft servers that are version 1.7 or higher.
        # Don't expect the player list to always be complete, because many servers run
        # plugins that hide this information or limit the number of players returned or even
        # alter this list to contain fake players for purposes of having a custom message here.
        status = server.status()
        print(f"The server has {status.players.online} player(s) online and replied in {status.latency} ms")

        # 'ping' is supported by all Minecraft servers that are version 1.7 or higher.
        # It is included in a 'status' call, but is also exposed separate if you do not require the additional info.
        latency = server.ping()
        print(f"The server replied in {latency} ms")

        # 'query' has to be enabled in a server's server.properties file!
        # It may give more information than a ping, such as a full player list or mod information.
        # query = server.query()
        # print(f"The server has the following players online: {', '.join(query.players.names)}")
        ```

    * <ins>Result</ins>:  
        ![mc_status](./mc_status.png)

> [!NOTE]
> The Minecraft server can be reached under the *IP address of your cloud VM on port 8888*: `http://IP_Address_VM:8888`.  
> The **ERR_EMPTY_RESPONSE** error message means that the browser tried to communicate with the server but did not receive any data. This **is to be expected** since a Minecraft Server doesn't send HTTP data that a browser can interpret. The Minecraft server uses Minecraft's own protocol, which is not compatible with a web browser.The Minecraft Server is running correctly and listening on the specified port (e.g. 8888), but it only responds to requests from Minecraft clients, not HTTP requests.  
> ![ip_address:8888](./ipaddress.png)

