# Docker Notes
Docker uses containers to run applications in isolated environments on a computer.

Without something like Docker, running multiple applications on a machine can require a considerable amount of setup and configuration, with different apps requiring different versions of the same language or dependencies.

A container packs up everything your app needs to run, from source code to dependencies to runtime environment. The container can run this app in isolation, away from any other processes on the machine, providing a predictable and consistent environment.

A virtual machine has its own full operating system and is typically slower, while containers share the host machine's operating system and are typically quicker.

## Images and Containers
Images are analogous blueprints for containers. They store:
- the runtime environment
- application code
- any dependencies
- extra configuration, such as environment variables
- commands

Images also have their own independent filesystem.

Images are read-only. To change them you need to create a brand new image.

Containers are runnable instances of images. When you run an image, it creates a container, a process that can run your app exactly as outlined in the image.

Containers are isolated processes, they run independently of any other process on the machine.

By sharing an image, multiple containers can be created and run on multiple machines.

## Parent Images and Docker Hub
Images are made up of several layers.

The parent image, including the OS and sometimes the runtime environment, is the first layer.

The next layers can be anything else you would add to your image, such as source code, dependencies and commands.

Docker Hub is an online respository of Docker Images. You pull images from Docker Hub using `docker pull IMAGE_NAME`. You can add tags to specify which version of an OS you want and which underlying OS.

If, for example, you need a specific version of Node.js, it's better to specify a version or Docker will use the latest version.

## Dockerfiles
To create your own image, you create a Dockerfile.

In general, each line in the Dockerfile represents a different layer in the image.

First, you specify which parent image to use, using `FROM`.

The which files you want to copy into the image and where to copy them to. Usually, you won't copy into the root directory to avoid clashing with other files.

You can specify a working directory for the image.

Next, specify what dependencies you want to install. You can specify what commands are run when the image is made using `RUN`. 

You also need a command to run the application in the container using `CMD`  and an array of strings in double quotes.

To communicate with the app, you also need to expose a port on the container using `EXPOSE`. This is used to set up port mapping.

```Dockerfile
FROM node:17-alpine

WORKDIR /app

COPY . .

RUN npm install

# required for docker desktop port mapping
EXPOSE 4000

CMD ["node", "app.js"]
```

To build the image use `docker build -t IMAGE_NAME .`

## .dockerignore
The `.dockerignore` file lets you specify any files or folders you want Docker to ignore when it copies files over to the image, for example node modules or logs.

## Starting and Stopping Containers
When you run an image from Docker Desktop, you have the choice to specify extra settings, such as:
- a container name
- ports to map to
- volumes

From the command line:

List images with `docker images`

The run an image and create a new container, you need either the name or id of that image: `docker run IMAGE_NAME/ID`.

Any options go before the image name/id:
- `--name CONTAINER_NAME` lets you name the container
- `-p PORT_ON_COMPUTER:PORT_EXPOSED_BY_CONTAINER` lets you map ports from your computer to the container
- `-d` runs the container in detached mode, leaving your terminal detached from the process

View running processes with `docker ps`

View all containers with `docker ps -a`

Stop a container with `docker stop CONTAINER_NAME/ID`

Restart a container with `docker start CONTAINER_NAME/ID` (you don't need to reconfigure the options)

## Layer Caching
Once an image is created, it's read-only. So if you make any changes to your application you'll need to create a new image.

Every time Docker builds an image, it stores each layer in a cache. When you build an image after this, before Docker starts the process it looks in the cache and tries to find a cached image that it can use for the new image it's creating.

If you have changed a layer, all layers built on top of that will be affected too.

If the dependencies haven't changed, however, it can make sense to copy over the ``package.json`` and run ``npm install`` before copying the files. This way the dependencies can be cached, speeding up build times.

```Dockerfile
FROM node:17-alpine

WORKDIR /app

# copy package.json to working directory
COPY package.json .

RUN npm install

COPY . .

EXPOSE 4000

CMD ["node", "app.js"]
```