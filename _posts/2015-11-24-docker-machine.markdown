---
layout: post
title:  "Docker Machine"
date:   2015-11-23 12:53:31
categories: docker part-6
topic: docker
---

A Docker Machine in simple words is a Docker host which has a configured client to use docker on that virtual host.

Docker Machine provides a number of commands to create, delete and manage various Docker hosts.

To install Machine, get the DOCKER TOOLBOX (for Mac and Windows Users) or follow the simple MACHINE INSTALLATION INSTRUCTIONS.

Docker Machine allows you to provision Docker on virtual machines that are either on your local system or a cloud provider.Docker Machine creates a host on a Virtual machine with docker engine installed on them so you can use docker on those VMs.

Lets look at the various Docker Machine commands.To go through them lets have a small example where we create two machines using virtualbox and manage both the machines.
Please install VIRTUALBOX to go with this example.

```
docker-machine create --driver virtualbox myhost1
```

The above command creates a VM called myhost1 with default preferences and memory allocations.
The *--driver* or *-d* flag is used to mention the driver used to create the vm.Typically on a local Linux, Mac or Windows system the driver is Oracle Virtual Box which i have used too. 
For cloud providers, Docker-Machine supports drivers such as AWS, Microsoft Azure, Digital Ocean and others. 

```
docker-machine ls
```

Lists the docker-machines that are present and their status.

Now, lets create another VM but limit its memory to 512 MB.

```
docker-machine create -d virtualbox --virtualbox memory 512 myhost2
```

```
docker-machine ip myhost1
```

returns the myhost1's IP address.

Now lets point our shell to the machine host1. Before that lets just run the env subcommand.

```
docker-machine env myhost1`
```

You will see a list of comamnds that point the docker environment variables to myhost1.To configure your shell to the above docker environment variables just run

```
eval "$(docker-machine env myhost1)"

docker info

```

You can see after entering docker info that its our shell is pointed to the docker host on myhost1 VM which uses bootdocker.

```
docker run hello-world

docker ps -a

eval "$(docker-machine env myhost2)"

docker ps -a

```

So now we have docker hosts running on different VMS, experient with them with respect to docker container networking.
For multi-hosy networking and managing clusters of docker containers, have a look at Docker Swarm.




