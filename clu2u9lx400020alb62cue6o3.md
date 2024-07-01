---
title: "Kubernetes Series Part 2 - Containerize a Node application"
seoTitle: "Kubernetes Series Part 2 - Containerize a Node application"
seoDescription: "With Docker, we can package the entire app into a portable image and then run that image on some other server."
datePublished: Mon Nov 21 2022 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clu2u9lx400020alb62cue6o3
slug: kubernetes-series-part-2-containerize-a-node-application
canonical: https://ilearnedathing.com/post/kubernetes-series-part-2-containerize-a-node-application
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122291010/8201b259-e71e-473b-b152-f9c38a5be4dc.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122331425/42581bff-1312-4cf5-a232-17534137255e.jpeg
tags: docker, nodejs, kubernetes

---

## What We’ll Build

Building upon my last post, we’re going to take our node application and turn it into a Docker image. We’ll ultimately be using Docker images in our kubernetes cluster later in the series.

With Docker, we can package the entire app into a portable image and then run that image on some other server. The image has its own little ecosystem so there’s no need to install all of the dependencies on the server for the app. As long as the server you’re wanting to use has Docker, you’re all set!

You can turn almost any application into a Docker image. You can even run an entire Postgres database in a container!

## Prerequisites

- Complete the first part of this series
- Docker installed on your machine. Check out the docs at [https://docker.com](https://docker.com) to get set up.

## Create the Dockerfile

The Docker file is the ‘blueprint’ for how an image is build. Let’s create a Dockerfile in the root of our project. As well as a .dockerignore file.

```bash
touch Dockerfile
touch .dockerignore
```

The `.dockerignore` file includes files and directories that we _do not_ want included inour image. Add the following to your `.dockerignore` file:

```yaml
# .dockerignore

node_modules
```

And in your `Dockerfile`:

```yaml
FROM node:18-alpine

RUN mkdir -p /usr/src/app

WORKDIR /usr/src/app

COPY . .

RUN npm install

EXPOSE 3000

CMD ['npm', 'run', 'start']
```

## What’s Happening Here?

```yaml
FROM node:18-alpine
```

Since this is a node application, we need node to run it. This line defines what image we will use to build our image. Alpine is a very lightweight Linux distribution - coming in at only 5MB.

```yaml
RUN mkdir -p /usr/src/app
```

`RUN` commands allow you to run linux commands during the build of your image. This command makes our working directory.

```yaml
WORKDIR /usr/src/app
```

`WORKDIR` defines the working directory of our application _inside_ the container.

```yaml
COPY . .
```

We want to copy everything from the directory where our Dockerfile is to the working directory of the container. The first period specifies the relative path locally. The second period specifies _where_ in the container we want those files copied.

```yaml
RUN npm install
```

Install the dependencies.

```yaml
EXPOSE 3000
```

The application binds to port 3000. This will let the docker daemon know where to bind the app.

```yaml
CMD ['npm', 'run', 'start']
```

`CMD` is the command that will be run when the container is ‘brought up’. There can only be _one_ `CMD` in a Dockerfile

## Build the Docker Image

We need to build the image so that we can push it to our Docker registry. In your terminal, run:

```bash
docker build -t <your-registry-name>/node-weather:latest .
```

`-t` is the image name and tag. The trailing `.` is saying that you want to build the Dockerfile in your present directory.

## Push the Docker Image

If you’re logged into Docker hub, run the following:

```bash
docker push <your-registry-name>/node-weather:latest>
```

You _may_ need to create an account and then login with `docker login`

## Run the Image Locally

Now let’s try it out! To run the image locally and start a container, run the following command:

```bash
docker run -p 3000:3000 <your-registry-name>/node-weather:latest
```

Visit `http://localhost:3000` and you’ll see your express server up and running.

That’s all 😃

## What’s Next?

In the next tutorial, we’ll jump into kubernetes. We’ll create a deployment for the image we just created in order to run a pod in a kubernetes cluster.
