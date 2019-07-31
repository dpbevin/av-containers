# The "Hello World!" of containers

## Prerequisites
Before you begin, you will need:
* Windows 10 (1803 or higher)
* [Docker Deskop](https://docs.docker.com/docker-for-windows/install/)

## Overview

In this example, we will:
1. Build an image
1. List the available images
1. Run the image
1. List the containers
1. Clean up the image

---

### Building your first image

To build your first docker image, open a Powershell (or command) prompt at this folder, then run the following command.
```
PS> docker build -t techtopics/hello-world .
```

The full stop instructs docker to look for a file called "dockerfile" in the current directory.

You should see the following:
> Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM mcr.microsoft.com/windows/nanoserver:1803
 ---> c015ed946ea6
Step 2/2 : CMD echo Hello World!
 ---> Running in 7d28fe6710a9
Removing intermediate container 7d28fe6710a9
 ---> 067139d248ae
Successfully built 067139d248ae
Successfully tagged techtopics/hello-world:latest

### Listing the available images

To list the available Docker **images**, run the following command.

```
PS> docker images
```
You will see at least these two images:
* techtopics/hello-world
* mcr.microsoft.com/windows/nanoserver

The helloworld image is the one we just build. The nanoserver image was automatically fetched from the Docker Hub as part of the build process.

### Running the Hello World image

To run the Hello World image as a **container**, run the following command.

```
PS> docker run techtopics/hello-world
```

You will see the output ```Hello World!```, then the container will exit.

### List the containers

Running the command ```docker ps``` shows you a list of **running** containers. Because our container exited, we need to run the command:

```
PS> docker ps -a
```

Note that the container has exited (status) but has a container ID: ```483a6854048b```

You can use this ID to reference the container in other docker commands (as shown below). Typically, you can use the first three digits of the container ID as a shortcut. Technically you only need the smallest subset of unique characters from the string.

### Delete the container

Delete the stopped container using the ```rm``` command followed by the container ID
```
PS> docker rm 483
```

As an alternative to deleting the container manually every time, you can pass the ```--rm``` argument into the ```docker run``` command like this:

```
PS> docker run --rm techtopics/hello-world
```
