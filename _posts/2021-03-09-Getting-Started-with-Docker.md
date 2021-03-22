---
toc: true
[layout](layout): post
description: Docker basics categories: [Docker] title: Getting Started in
Docker 
---
# Getting Started with Docker

## Introduction

Docker is an excellent tool for working with multiple development environments. Often the need arises that you wish to build a new package in python or another language and realise the dependencies are all wrong. Not to mention you need to install underlying system packages to get this up and running. Docker fills this space by providing seperated containers to silo your applications. Docker can be viewed as a virtual machine if you wish to visualise it in some way. Docker sits on top of an underlying system and utilises the systems resources. Much like a Virtual machine. The big advantage of Docker over a virtual machine is that it doesn't require startup because it uses the Kernel (OS instructions) provided by the host machine. This reduces setup and initialisation time for each container as the system is already running. The Docker philosophy is to build containers that run one specific task or service, such as a web server or a database.

# Installation
Docker has many powerful features, some of these will be outlined in the tutorial below. To begin with we shall install Docker using your system package manager. As much of the system development i do is on a Jetson board with a host Ubuntu these commands are tailored to the Ubuntu commands but are easily swapped for arch or fedora etc. The command to run for installation is 
```
sudo apt install docker
```

Once docker is installed we can begin playing with it. The first thing we can do is remove the need for docker to require sudo priviledges to run. This allows us to develop and run the docker containers without having to call sudo at each docker command. This can be revoked at a later date if you need to harden your system. The command is:
```
sudo usermod +aG docker $USER
```

# First steps

This will add the current user denoted by the `$USER` indicator to the docker group. Once this is done we can enable this by restarting the terminal.

Now that we have docker setup we can pull down a basic container to test whether the system is working. To do this we will pull down the base ubuntu image. This image is large but can be utilised later as a base for your own containers so the download is not wasted, the command to run is 
```
docker pull ubuntu:latest
```

The latest statement after the semicolon is to retrieve the latest ubuntu version on docker hub. This dwnload will take a bit to run. Once it is complete we can verify we have the docker image by running:
```
docker images
```

This will list all the downloaded docker images. If we wish to remove these images we can run:
```
DO NOT KNOW THE COMMAND
```

We can see the Ubuntu image is available and we can start the docker container with:
```
docker run -it ubuntu:latest
```

We have two flags here -i and -t which can be combined as they are shown here to increase readability. The i flag indicates interactive and the -t indicates terminal. We can confirm the docker container is running as our prompt will change from the system location to 
```
root@<alphanumeric_string>
```

We can use many of the commands that would be used in a standard linux system including `cd`, `ls`, `mkdir`. However we can note that the default docker containers are setup with root access and no additional user. We will fix this later in the tutorial.

We can exit out of the docker container with `ctrl -d`. The major feature of a docker container is that the system is non persistent so whenever the container is exited no state is saved. For instance if you ran `apt install wget` in your instance then closed the docker container. Newt time you ran the container you would not have this package. We can fix this using a dockerfile. An introduction to dockerfiles will be shown below. Before we get there we can do some more things in prebuilt containers with out an issue. One thing we may want to do is utilise a docker container such as ```https://hub.docker.com/r/jupyter/datascience-notebook/```. This container is a Jupyter notebook stack for running datascience projects. 

We can download it with 
```
docker pull jupyter/datascience-notebook
```

The cool thing about this container is we can port forward (send to the main machine) the jupyter notebook. The host computer is then able to run the jupyter notebook in the browser just like normal. The difference is that you are on a different machine. An issue that occurs with our non persistant docker containers is that we can no longer save files in the container for use at another stage. How do we get around this. In docker we have another flag known as `-v` or volumes. We setup our run script as shown, using the previously downloaded datascience docker image. To allow access to the notebook on the host machine we also use the `-p` flag and forward port `10000:5000```
```
docker run --rm -p 10000:5000 -e JUPYTER_ENABLE_LAB=yes -v "$PWD":/home/jovyan/work jupyter/datascience-notebook:9b06df75e445
```

The command above points the host folder found in /home/<my_python_folder> to the destination or docker container location ~/<python_folder>. Thus allowing us to save any of our python scripts in a persistant and accessible folder for later. Docker also allows us to create volumes external to docker and point to these in the container thus seperating host and client files.

## Building your own containers

