---
toc: true
layout: post
description: Docker setup for working with different cpu architectures
categories: [Docker, Jetson, QEMU]
title: Docker cross compiling on x86 for ARM
---

# Configuring Docker for use on Jetsons

## Making Docker capable of compiling ARM containers using QEMU


We need to install and configure qemu to run docker containers with our main machines.

`sudo apt-get install qemu binfmt-support qemu-user-static # Install the qemu packages`

`docker run --rm --privileged multiarch/qemu-user-static --reset -p yes # This step will execute the registering scripts`

`docker run --rm -t arm64v8/ubuntu uname -m # Testing the emulation environment`
`aarch64 # Outputs the correct architecture`





## Add docker to user group

`sudo usermod -aG docker $USER`


## Configure docker for NVIDIA
`distribution=$(. /etc/os-release;echo $ID$VERSION_ID)`
`curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -`
`curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list`

`sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit`
`sudo apt install -y nvidia-docker2`
`sudo systemctl daemon-reload`
`sudo systemctl restart docker`




References 

https://www.stereolabs.com/docs/docker/building-arm-container-on-x86/

https://dev.to/caelinsutch/running-docker-containers-for-the-nvidia-jetson-nano-5a06
