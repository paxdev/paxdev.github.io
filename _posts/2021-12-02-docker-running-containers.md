---
  title: Running Docker containers
  header:
    subheading: Getting started with Docker part 1
    backgroundImage: /assets/bg/docking.jpg
    overlay: true
  categories:
    - getting-started-with-docker
  tags:
    - docker
---

This is the first in a series of posts giving a high-level step-by-step view of getting started with Docker.

In this post we'll look at running containers and interacting with them. 

I'm assuming a familiarity with what `Containers` are and why you may want to use them. If not [Red Hat have an introduction here](https://www.redhat.com/en/topics/containers/whats-a-linux-container). 

## Pre-requisites

This post assumes that you already have [Docker installed](https://docs.docker.com/get-docker/).

## Container Registries

We need somewhere to get container images from.

A `Container Registry` is a collection of `Repositories` used to store `Container Images`.

It's essentially the equivalent of `npm` or `NuGet` for containers.

The biggest registry is [https://hub.docker.com](https://hub.docker.com). The default `Docker` installation is already configured to work with this registry out of the box.

_Note that there is a download limit for images. 
You can increase the number of images you are able to download by registering for a free account and there are paid business plans with higher/unlimited image pulls._

_Once an image is pulled it is stored locally and does not to be downloaded again._

### What images can I run?

For the purposes of this demo I am using the web server Nginx as it's a useful way to illustrate the features I want to discuss.

More useful ways I typically use Docker at home are when I want to try something complicated with a lot of dependencies such as specific versions of Java, Node, .Net etc.

Great examples are things like TeamCity, Jenkins, Build Agents for Azure DevOps, Octopus Tentacles, Ansible. 

You can fire up a Docker image which has already configured these dependencies for you, and, when you quit the container, you can just delete the image and container and your local environment is left unchanged. 

At it's most basic, of course, it can be just a simple way to run a Linux tool on a Windows box.

## Pull an Image

Let's pull our first image

```
docker pull nginx
```

Once the image is pulled, we can list images 
```
docker image ls

REPOSITORY       TAG       IMAGE ID       CREATED      SIZE
nginx            latest    f652ca386ed1   7 hours ago   141MB
```

## Specific versions of Images

Images are specified by a `Tag`. In the above example ew have the `latest` image. 

We can pull a specific version of an image as follows 

```
docker pull <<image-name>>:<<tag>>
```

_Note that the `latest` tag can be misleading. See [this article](https://vsupalov.com/docker-latest-tag/) for an excellent explanation._

## Run a Container 

The image is a blueprint for creating a `Container` (similar to a `Class` being a blueprint for creating an `object`).

In order to do anything useful we need to spin up an instance of our `Image` in a `Container`

We use the `docker run` command to spin up an instance of our `container`. 

At it's most basic the command takes the form

```
docker run <<image-name>> <<command-to-run>>
```

If we don't pass any additional flags, the command will run to its end. For now we want to run an interactive session, so we'll pass the `-i` flag, and to enable us to set up a terminal we'll pass `-t`.

Let's spin up a `bash` terminal in `nginx`.

```
docker run -it nginx /bin/bash
```

This will result in us running an interactive `bash` session inside our container. 

There is nothing to stop us from starting up multiple instances of the same `image`.

We can list all our `containers` using (in a different terminal window!)

```
docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS          PORTS     NAMES
aad0fd572eb4   nginx     "/docker-entrypoint.…"   5 minutes ago   Up 10 seconds   80/tcp    quirky_curie
```

Note that unless you specify a name using the `--name` parameter, `docker` assigns a random name to your container.

I can stop my container by `exit`-ing the bash session, or I can use `docker stop <<name>>`.

Most `docker` commands also accept partial IDs (enough to uniquely identify an artifact), e.g.

```
docker stop quirky_curie
docker stop aad0
```

I can list all containers (including stopped ones) using 

```
docker ps -a

CONTAINER ID   IMAGE                COMMAND     CREATED          STATUS                        PORTS     NAMES
daa389d05735   nginx                "/bin/bash" 2 minutes ago    Exited (0) 13 seconds ago               quirky_curie
```

I can remove containers using `docker rm <<name or partial ID>>` and I can clean up all stopped containers with `docker container prune`.

I can stop and start containers using `docker start/stop <<name or partial ID>>` and I can interact with a running container using `docker exec -it <<name or partial ID>> <<command>>`

Images can be removed using `docker rmi <<name>>`. Images can also be pruned using `docker image prune -a` to remove all images that are not linked to a container.

**NB** 
* If you issue a `docker run` command on an image that you do not have locally `docker` will try to download it for you 
* If you want the container (but not the image) to be immediately cleaned up on exit use the flag `--rm`
* Find more help with, e.g. `docker run --help`

## Interacting with the container

In order to interact usefully with the container we need a way to help it interact with the outside world.

### Expose Ports

We use the `-p host-port:container-port` parameter to expose ports on the container to the host OS. Running ...

```
docker run --rm -p 8080:80 nginx
```

... tells Docker to expose port 80 on the container to port 8080 in our host system. We can now navigate to `http://localhost:8080/` to see the default `nginx` launch page and verify that `nginx` is indeed running.

We can stop the container docker ps

### Mount a volume

I can use the `-v host-dir:container-dir` parameter to mount a folder in my local OS to one in the container allowing us to pass files to/from the container images.

```
# In the host
docker run -v c:\Users\demo:/mnt/demo --rm -it nginx /bin/bash

# In the container
ls /mnt/demo
```

### Set environment variables

I can set environment variables using `-e VARNAME1 -e VARNAME2=value`

```
# In the host
docker run --rm -e MYVAR=hello -it nginx /bin/bash

# In the container
echo $MYVAR
hello
```
