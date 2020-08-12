---
title: Dockerizing My Dev Environment
date: 2016-10-01 23:48:57
tags: ['Docker', 'Node.js', 'Express']
categories: ['Docker', 'Node.js', 'Express']
---

[Docker](https://www.docker.com/) is a containerization platform that simplifies building, shipping and running apps.

As a developer, having a consistent environmment removes a lot of the pain associated with having to make sure the computers you use have the same version of software such as frameworks e.g. node. 

There many more benefits docker offers and, environment consistency and the ability to have other developers run your app without having to configure the environment (apart from having docker installed) is what this post is about. 

I'm going to demonstrate how to dockerize a Node (Express) app for development.

I'm going to assume you have: 
 * An app generated using [express-generator](https://expressjs.com/en/starter/generator.html)
 To generate:

 ``` bash
 $ npm install express-generator -g
 $ express dockerize_dev_env --hbs
 ```
 * Docker installed on your machine: [Docker for Mac or Windows](https://www.docker.com/products/docker)

With our app ready and running using node on the local machine, we can now change it such that our app should run in a docker container with node and anything else we need for development installed on it. This means that we could run this container with docker on another machine without needing to have node on installed on it.


## The Dockerfile
We first create the Dockerfile at the root of the project generated (above).
This file is what is used to build the node image for our environment.
Read more about the commands used in the file [here](https://docs.docker.com/engine/reference/builder/).

``` dockerfile
FROM node:latest

MAINTAINER DMKCode

ENV NODE_ENV=development 
ENV PORT=3000

COPY . /src
WORKDIR /src

# use nodemon for development
RUN npm install --global nodemon

RUN npm install

# we do this because when we create the container using this image, node_modules from npm install is wiped out as the volume is mounted when creating the container.
RUN mv /src/node_modules /node_modules

EXPOSE $PORT

CMD ["npm", "start"]
```
Essentially what the file does is pull in the latest node image to use as the base image, then sets who the maintainer of the image is i.e. DMKCode. 
The file then defines two environment variables which will be available to the container created from the image for the app in the container to use. 
The file then copies the source code from the root folder locally into the image in a folder /src which is also then set as the working directory.
The file then installs [nodemon](https://github.com/remy/nodemon) (watches files and restarts the application if there is a change) globally in the image, then installs the dependencies defined in package.json and then moves the node_modules folder.
Finally, the file then specifies the default executing command for the container created from the image.

We then create the [docker-compose.yml](https://docs.docker.com/compose/) which is particularly useful for multi-container Docker applications. We will use it here for linking the local source to a volume which will be mounted on a container, and naming our container and image without using the command line. 

## The docker-compose.yml
``` yaml
version: "2"

services:
  node_express:
    container_name: node-express
    image: dmkcode/node-express
    build: 
      context: .
    command: nodemon --debug=5858
    volumes:
      - .:/src
    ports:
      - "8000:3000"
      - "5858:5858"
```
The file defines a service called node_express from the image built from the Dockerfile in the current directory.
The file, for develpment overrides the command specified in the Dockerfile which enables remote debugging provided by node. However, the image on its own will not have this enabled which is suitable for production. For more about live debugging, check out the [blog](https://blog.docker.com/2016/07/live-debugging-docker/) by [Aanand Prasad](https://blog.docker.com/author/aanand/).
The file then mounts the current directory as a volume /src overwriting anything in it. This means the image will not have to be rebuilt every time there is a change to the source code. 
The file then maps port 3000 in the container to port 8000 externally i.e. machine localhost and port 5858 in the container to port 5858 externally i.e. machine localhost for connecting to the remote debugger.

## Run It
Check out the app by running: 

``` bash
$ docker-compose build
$ docker-compose up
```

Go to http://localhost:8000 to see the app!

{% asset_img node_express.png 100% node app %}
<br>
If you were to make changes to the source code, the app in the container will be automatically restarted by nodemon meaning no need to rebuild the image. 

The complete source code can be found on my [Github](https://github.com/DMKCode/dockerize_dev_env). 

Thanks for reading!