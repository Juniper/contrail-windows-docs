# Docker

Docker is a popular tools for managing containers on Linux.

Since Windows Server 2016, docker is the main tool used for managing Windows Containers.

## Docker client

Docker command line tool, which communicates with Docker Daemon over a named pipe.

## Docker daemon

A program that runs as a service. It exposes an API over a named pipe. It's responsible for managing containers, from parsing Dockerfiles and pulling layers to basic networking.

Docker is cross-platform and is integrated into many different network orchestration systems. This is achieved thanks to its plugin architecture. One can implement all kinds of plugins for docker, for example for volume or network management. These plugins can either be integrated into docker's code directly, or can be implemented as Remote drivers. The advantage of Remote drivers is that one doesn't have to build the whole Docker if she wants to implement a driver. Remote drivers on Windows can use tcp sockets or named pipes for communication. In either case, docker daemon will look for a "spec file" in a specific path: $Env:ProgramData\docker\plugins\. The name of the spec file must be the same as driver's name. The file itself contains a protocol and address on which docker daemon will try to initiate communication.

Since docker is written in Go, Microsoft engineers had to expose an API for managing Windows Containers written in this language. 

## How to run docker daemon in debug mode

Just run dockerd.exe -D in console or add "debug": true to config and restart daemon. This will enable debug logging.

# Host compute service (HCS) & host network service (HNS)

Golang wrapper for HCS and HNS is implemented in a public hcsshim repository: www.github.com/Microsoft/hcsshim. HCS and HNS are Windows Services responsible for light container virtualization as well as setting up virtual networking for them. They are implemented in some system dynamic libraries, which interact with the kernel directly. How it works isn't documented.

## Host compute service

Deals with volumes, filesystem layers etc. It's related to HNS, but it's not inside scope of the project directly.

## Host network service

HNS is a wrapper for syscalling functions in vmcompute.dll. Only one file in the whole hcsshim repository is used by HNS - 'hnsfuncs.go'.

# Windows Containers

Windows Containers are a feature shipped with Windows Server 2016, Windows 10 and Windows Nano Server. These act similarly to lxc containers. On Windows, they are implemented using various namespaces, as well as network namespace equivalent (called network compartment).

Network compartments are not documented, but vmcompute.dll contains some functions related to how they work.
Unlike on Linux, Windows supports two types/levels of isolation for its containers:
* Windows Server containers, which work just like normal Linux containers - they all share the kernel
* Hyper-V containers. These are essentialy small VMs (running Nano Server), that run Windows Containers which then run the container we wanted. This additional level of virtualization is useful mostly for security reasons. It provides some pros and cons of both containers and VMs. To run a container in this mode, one needs to add "--isolation-level=hyperv" to "docker run" command.

There are no differences on how networking is implemented for both of these types of containers.

There are two base images for docker container creation:

* microsoft/windowsservercore which is headless version of normal Windows Server Core
* microsoft/nanoserver which is very lightweight, stripped of most functionality version of Windows Server Core
