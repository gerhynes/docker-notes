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

```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
docker-compose*
```

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

## Managing Images and Containers
`docker images` lists all the images you have.
`docker processes` lists all the running containers.
`docker ps-a` lists all containers.
`docker image rm IMAGE_NAME` deletes an image (if it isn't being used by a container).
`docker image rm IMAGE_NAME -f` will delete an image even if it being used by a container.
`docker container rm CONTAINER_NAME` deletes a container.
`docker system prune -a` will remove all containers, images and volumes.

In Docker, versions are denoted by tags, letting you create multiple versions of images with certain variations.

To create an image with a tag, use `docker build -t IMAGE_NAME:TAG .`

To run a container for a specific image version, specify the tag `docker run --name CONTAINER_NAME -p 4000:4000 IMAGE_NAME:TAG` 

## Volumes
`docker run` will always run the image via a new container. `docker start` will run an existing container. 

If you stop a container, make changes to the app within it, and restart the container it won't reflect changes made to the app. This is because the image in the container exists and once an image is made it becomes read-only. To see the changes to the app, you need to create a new image and run that image in a new container.

Volumes are a way around this. Volumes let you specify folders on your host computer that can be made available to running containers. You can map these folders on your host computer to specific folders in the container so that if something changes in the folders on your computer, those changes would also be reflected in the container.

If you mirror the root folder of a project on your host computer to the working directory of the container, you would see every update without having to build a new image.

Importantly, the image itself does not change. Volumes just map directories between a container and the host computer. If you to update the image to share it or use it to create new conatiners, you'd have to recreate the image using `docker build`. Volumes are useful during development and testing.

To set up a volume, use the `-v` flag, an absolute path to the directory on the host computer and an absolute path to the directory in the container.

```
docker run --name myapp_c_nodemon -p 4000:4000 --rm -v C:\Users\Gerard\Desktop\docker-crash-course\api:/app myapp:nodemon
```

If you want to prevent a particular directory in the container form being mapped to the host computer (for example, to keep the node_modules from being deleted/changed) you can use an anonymous volume. This maps the directory in the container to a directory managed by Docker. It will override the previous mapping as its path is more specific.

```
docker run --name myapp_c_nodemon -p 4000:4000 --rm -v C:\Users\Gerard\Desktop\docker-crash-course\api:/app -v /app/node_modules myapp:nodemon
```

## Docker Compose
Sometimes you might want multiple programmes to run at once in separate containers and communicate with each other (for example, an API, database and frontend). 

Docker Compose is a tool built into Docker that lets you make a single `docker-compose.yaml` file that contains all the container configuration of your project. 

Make the Docker Compose file in the root directory shared by all the projects you want to run in tandem.

Docker Compose creates the image and runs the container for it.

Each container you want to refer to is a `service`.

```yaml
version: "3.8"
services:
	api:
		build: ./api
		container_name: api_c
		ports:
			- "4000:4000"
		volumes:
			- ./api:/app
			- ./app/node_modules
```

To run the Docker Compose file, use `docker-compose up` (with `-d` if you want to run it in detached mode).

To stop and delete the container while keeping the images and volumes, use `docker-compose down`.

To remove all images and volumes, use `docker-compose down --rmi all -v`.

With Docker, every service (whether it's an API, a database or a frontend application) runs in an independent, isolated container.

If you run `docker-compose up` again, it won't rebuild the image. You need to tell it if you've made changes and want to rebuild the image.

`docker-compose up --build` will force a rebuild of the image.

## Sharing Images on Docker Hub
To share an image on Docker Hub, go to `hub.docker.com` and sign in.

Select `Create a Repository` and give it a name. From your terminal, use `docker login` to log into Docker and then use `docker push REPOSITORY_NAME:TAG_NAME` to push the image to Docker Hub. 

Use `docker pull REPOSITORY_NAME:TAG_NAME` to download the mage from Docker Hub.

## docker exec
`docker exec` lets you run commands in a running container. For example, you can execute an interactive bash shell on the container.

```bash
# docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

docker exec -it node-docker_node-app_1 bash
```

## Development vs Production Configs
You can set up multiple `docker-compose` files for different environments, such as `docker-compose.dev.yml` and `docker-compose.prod.yml`. Any configuration that is shared between every environment should go in a main `docker-compose` file.

```yml
# docker-compose.yml
version: "3"
services:
  node-app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
```

```yml
# docker-compose.dev.yml
version: "3"
services:
  node-app:
    volumes:
      - ./:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: npm run dev
```

```yml
# docker-compose.prod.yml
version: "3"
services:
  node-app:
    environment:
      - NODE_ENV=production
    command: node index.js
```

You can point to multiple `docker-compose` files using the `-f` (file) flag. Order matters, as the files you pass in will be run in order.

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

You can then stop them with `docker-compose down`.

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml down -v
```

You can tell your Dockerfile whether you're running in a development or production environment using an embedded bash script in your `package.json` and by passing the `NODE_ENV` argument in `docker-compose`.

```Dockerfile
FROM node:16

WORKDIR /app

COPY package.json .

ARG NODE_ENV

RUN if [ "$NODE_ENV" = "development" ]; \
      then npm install; \
      else npm install --only=production; \
      fi

COPY . ./

ENV PORT 3000

EXPOSE $PORT

CMD ["node", "index.js"]
```

```yml
# docker-compose.prod.yml
version: "3"
services:
  node-app:
    build:
      context: .
      args:
        NODE_ENV: production
    environment:
      - NODE_ENV=production
    command: node index.js
```

## Working with Multiple Containers
If you need to run an additional container as part of your application, for example a database, you can add this as a service in your `docker-compose`.

Make sure to pass in any environment variables it requires.

```YAML
version: "3"
services:
  node-app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
  mongo:
    image: mongo
    environment:
      - MONGO_INITDB_ROOT_USERNAME=ger
      - MONGO_INITDB_ROOT_PASSWORD=mypassword
```

You can connect to the Mongo shell on the MongoDB container using `docker exec -it name_of_container mongo -u "username" -p "password"`.

If you run `docker-compose down` you'll delete the container, including the database.

You use named volumes to save this information. A named volume can be used by multiple services, so you need to declare it at the bottom of you `docker-compose` file.

```YAML
		volumes:
      - mongo-db:/data/db #named volume
     
volumes:
  mongo-db:
```

When you use `docker-compose down` don't include the `-v` flag or it'll delete the container.

### IP Addresses
Docker automatically assigns an IP address to each container.

Use `docker inspect container_name` to view (a lot of) data about your container. Under `Networks`, you'll see `IPAddress`.

You can use `docker logs container_name` to connect to a container and have access to its logs.

Docker makes it relatively easy to communicate between containers on the same custom network. When you have a custom network, you have DNS.

Use `docker network ls` to view networks. `bridge` and `host` are the defaults that come with Docker.

You can refer to a container's IP address based off it's service name.

```JavaScript
// IP address
mongoose.connect("mongodb://ger:mypassword@172.25.0.2:27017/?authSource=admin")

// service name
mongoose.connect("mongodb://ger:mypassword@mongo:27017/?authSource=admin")
```

Use `docker network inspect custom_network_name` to get more information on the custom network.