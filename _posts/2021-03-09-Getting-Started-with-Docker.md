---
toc: true
layout: post
description: Docker basics 
categories: [Docker] 
title: Getting Started in Docker 
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

There comes a time when you want to add certain features to your container that is not inbuilt in the image for example you may want to implement some machine learning and your python container only contains the main python libraries. You don't want to run pip install torch everytime you want to do anything. To overcome this we can build a dockerfile. This is basically a recipe for building a docker image. Below is an example docker image to implement machine learning using pytorch.


```
## Taken from https://github.com/anibali/docker-pytorch  ::: Many other images around with similar setups
FROM nvidia/cuda:11.0-base-ubuntu20.04

# Install some basic utilities
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    sudo \
    git \
    bzip2 \
    libx11-6 \
 && rm -rf /var/lib/apt/lists/*

# Create a working directory
RUN mkdir /app
WORKDIR /app

# Create a non-root user and switch to it
RUN adduser --disabled-password --gecos '' --shell /bin/bash user \
 && chown -R user:user /app
RUN echo "user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-user
USER user

# All users can use /home/user as their home directory
ENV HOME=/home/user
RUN chmod 777 /home/user

# Install Miniconda and Python 3.8
ENV CONDA_AUTO_UPDATE_CONDA=false
ENV PATH=/home/user/miniconda/bin:$PATH
RUN curl -sLo ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-py38_4.8.3-Linux-x86_64.sh \
 && chmod +x ~/miniconda.sh \
 && ~/miniconda.sh -b -p ~/miniconda \
 && rm ~/miniconda.sh \
 && conda install -y python==3.8.3 \
 && conda clean -ya

# CUDA 11.0-specific steps
RUN conda install -y -c pytorch \
    cudatoolkit=11.0.221 \
    "pytorch=1.7.0=py3.8_cuda11.0.221_cudnn8.0.3_0" \
    "torchvision=0.8.1=py38_cu110" \
 && conda clean -ya

# Set the default command to python3
CMD ["python3"]
```

Much of the code can be read as standard linux commands with an added docker recipe command. We can run through the dockerfile step by step to further flesh out the commands. If you wish to skip over this section jump to ###HYPERLINK TO END###.

The first command is 
```
FROM nvidia/cuda:11.0-base-ubuntu20.04
```
This is pointing the docker image to a base image which in itself is a compiled docker image from another source. It is possible to run a bare minimum docker image calling `FROM scratch` which will have nothing installed and you add everything you need to the image. We often want the functionality of a previously designed base for certain reasons in this case we are pulling from nvidias cuda build saving us the time of installation of CUDA in our own container.

Each docker command creates a layer allowing you to roll back or change setup without having to redownload from the start. 

The next steps are to install basic utilities and create and move into a working directory.

```
#install some basic utilities
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    sudo \
    git \
    bzip2 \
    libx11-6 \
 && rm -rf /var/lib/apt/lists/*

# Create a working directory
RUN mkdir /app
WORKDIR /app
```

The command RUN tells the docker container to run a linux command and WORKDIR sets the current folder.

We then can add and modify users using 

```
# Create a non-root user and switch to it
RUN adduser --disabled-password --gecos '' --shell /bin/bash user \
 && chown -R user:user /app
RUN echo "user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-user
USER user
```

We can begin to see that the commands are exactly like you would do in setting up your own install on  a machine. The next steps create the home directory, add environment variables and install Miniconda with a python environment.

```
#All users can use /home/user as their home directory
ENV HOME=/home/user
RUN chmod 777 /home/user

# Install Miniconda and Python 3.8
ENV CONDA_AUTO_UPDATE_CONDA=false
ENV PATH=/home/user/miniconda/bin:$PATH
RUN curl -sLo ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-py38_4.8.3-Linux-x86_64.sh \
 && chmod +x ~/miniconda.sh \
 && ~/miniconda.sh -b -p ~/miniconda \
 && rm ~/miniconda.sh \
 && conda install -y python==3.8.3 \
 && conda clean -ya
```

We can string commands together with the && command create a single layer following a set install stage. It is of note that if these become to large if for what ever reason the installation fails it will fall back to the previous known layer. For example if the miniconda installation fails the system will revert to the layer where the environment variables are set.

Finally the dockerfile installs torch and additional utilies before cleaning up with:
```
# CUDA 11.0-specific steps
RUN conda install -y -c pytorch \
    cudatoolkit=11.0.221 \
    "pytorch=1.7.0=py3.8_cuda11.0.221_cudnn8.0.3_0" \
    "torchvision=0.8.1=py38_cu110" \
 && conda clean -ya
```

The last command in a dockerfile is usually CMD to execute a program at each docker run command. In this situation we have 
```
CMD ["python3"]
```


## Going further

But wait if we are installing all these packages how did we get access to a jupyter notebook. We can setup a nice jupyter lab environment with the below code. This can be placed as the last block in your dockerfile to start the Jupyter lab.


```
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
    apt-get update && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean && \
    pip3 install jupyter jupyterlab==2.2.9 --verbose && \
    jupyter labextension install @jupyter-widgets/jupyterlab-manager
    
RUN jupyter lab --generate-config
RUN python3 -c "from notebook.auth.security import set_password; set_password('', '/root/.jupyter/jupyter_notebook_config.json')"

CMD /bin/bash -c "jupyter lab --ip 0.0.0.0 --port 8888 --allow-root &> /var/log/jupyter.log" & \
	echo "allow 10 sec for JupyterLab to start @ http://$(hostname -I | cut -d' ' -f1):8888 (password nvidia)" && \
	echo "JupterLab logging location:  /var/log/jupyter.log  (inside the container)" && \
	/bin/bash
```

## Outside of Software Development

There are many other things we can do with docker containers outside of software development. Below are a couple of examples of things we can do in a docker container. 

- https://hub.docker.com/r/linuxserver/plex  -- A plex server
- https://hub.docker.com/r/linuxserver/heimdall -- Web Application /Docker image server
- https://hub.docker.com/r/pihole/pihole -- A Network wide ad blocker
- https://hub.docker.com/r/linuxserver/lychee -- Photo management tool
- https://hub.docker.com/r/linuxserver/nextcloud -- file storage similar to google drive.
- https://hub.docker.com/r/linuxserver/pylon -- IDE built with node.js (strictly software development but a web based containerised development environment.
- https://hub.docker.com/r/rancher/rancher -- Container management platform
- https://awesomeopensource.com/project/gcgarner/IOTstack -- Full IOT stack


## Getting in and out of containers, and shutting down a docker service

### Entering

We often wish to run docker as a service and only access the environment when we need to. We can jump into a currently running docker image with the docker exec command:
```
docker exec -t CONTAINER bash
```
CONTAINER is a currently running image.

### Exiting

We can exit out of docker containers without terminating them by using the `-d` flag during initialisation to put the container in a detached mode.

### Shutdown
 We can shutdown a container using the kill command, gracefully shutting down the service
 
 ```
 docker kill --signal=SIGTERM <containerId>
 ```
 

## Final Notes

Docker has many potential applications and features and we have barely scratched the surface of its full potential. We can see from this short overview that the container design makes for an excellent development environment with both interactive and non interactive setups.



