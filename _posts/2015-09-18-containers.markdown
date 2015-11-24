---
layout: post
title:  "Containers"
date:   2015-09-17 12:53:31
categories: docker part-1
topic: docker
---

##Containers

Containers are basically light weight VM. But unlike traditional virtualization, they do not require an emulation layer to be run on but can use the Operating system's normal system call interface which reduces overhead and allows a large number of containers to be run on a host.

Using docker containers can be built very fast.

Containers 
  
  * have their own process space.
  
  * have their own network interface.
  
  * can run as root.
  
  * share the kernel with the host.

This means that Containers are Isolated processes and for isolation they rely on two major kernel features which are namespaces and controlgroups.


<hr>

#### 1. Namespaces
  * **pid**
  
  Each container has its own pid number and ps

  * **mnt**

  Mount namespace

  * **net**

  The network namespace  <br>	
  Each container has its own ip address assigned, own network interface and routing tables.
    
  * **uts**

  Containers have their own uts namespace , each one has a different hostname.

<hr> 

#### 2. ControlGroups

Incharge of accounting and limiting the system resources for containers.

  * **Memory** 

Limits memory for each container.
  
  * **Cpu** 

Prioritize containers,keep track of system CPU time
  
  * **blkIO** 

Block IO to keep track of Ios for each block device (limit the amount of data that can be written by the container) 
  
  * **Devices** 

Controls read/write/mknod permissions

<hr>

####	RUNNING CONTAINERS



 Containers can be run using the following command
 {% highlight sh %}
  $ sudo docker run --name my_container -i -t ubuntu bin/bash
  {% endhighlight %}

where,
	 *my_container*    ->  name of the container<br>
	 *ubuntu*    	   ->  the image<br>
     */bin/bash* 	   ->  the command to be executed<br>

Command line flags that can be used with run command are:
	
**-i** for STDIN<br>	
**-d** to detach and run container in background<br>
**-t** for pseudo-tty<br>
**-u** to specify a user the image should be run as<br>
**-e** to set environment variables during runtime<br>

<hr>
 Containers can be listed using the 
  
  {% highlight sh %}
  $ docker ps 
  {% endhighlight %}
 
 command which lists the running containers and -a can be used to list all containers and -q to return container Ids as shown below.

{% highlight sh %}
  $ docker ps -a -q
  {% endhighlight %}	

<hr>

 Containers can be stopped using the command
  {% highlight sh %}
  $ docker stop container_id
  {% endhighlight %} 
or
  {% highlight sh %}
  $ docker kill container_id
  {% endhighlight %}	



*container_name can be used instead of container_id as well.
  
 They can be started again using
 {% highlight sh %}
  $ docker start container_id
  {% endhighlight %} 
  
 and can re-attach to the previous session using
 {% highlight sh %}
  $ docker attach container_id
  {% endhighlight %}

<hr>  

 Container details such as id, network settings, mounts, volumes etc can be retrieved using the following command
 {% highlight sh %}
  $ docker inspect container_id 
  {% endhighlight %}
  
  {% highlight sh %}
  $ docker stats container_name1,container_name2 
  {% endhighlight %}

<hr>  

 Deleting a container
 {% highlight sh %}
  $ docker rm container_id 
  {% endhighlight %}
  

$ docker run --rm --volumes-from pg_test -t -i busybox sh


<a href="introduction-to-docker.html"><<Introduction</a> 
<a style = "float:right" href="images.html">Images>></a> 

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
