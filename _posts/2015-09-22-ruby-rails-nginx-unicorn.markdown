---
layout: post
title:  "Rails on Nginx using Unicorn"
date:   2015-09-17 11:05:32
categories: docker part-3
topic: docker
---

A Rails application is usually deployed on nginx using *unicorn* or *puma* rather than using development servers such as *Webrick* or *thin*.
To develop the docker way, our goal is to test and build our application in a production similar environment so that they can be deployed easily.

#### Unicorn

Unicorn is an HTTP server for Rack applications designed to only serve fast clients on low-latency, high-bandwidth connections and take advantage of features in Unix/Unix-like kernels.

Unicorn follows the Unix philosophy, load balancing in Unicorn is done by the OS kernel and Unicorn’s processes are controlled by Unix signals.More about unicorns design is given here.

Load balancing between worker processes is done by the OS kernel. All workers share a common set of listener sockets and does *non-blocking accept()* on them. The kernel will decide which worker process to give a socket to and workers will sleep if there is nothing to accept().

#### Rails on Unicorn

In our example,

First, we set up nginx in front of Unicorn, then we have rails application on unicorn, which interacts with a postgres database.
So we should have two containers, one for the rails apllication on nginx and another for the postgres database. To make things faster, we add our ruby gem cache in a separate container so that bundle install is much faster.

So let's break it up into steps:

###### Dockerfile for rails application

The Dockerfile for building the rails application would be as follows:

	#Base Image
	FROM ruby:2.2.0

	RUN apt-get update

	#Install nodejs
	RUN apt-get install -qq -y nodejs

	#Install Nginx.
	RUN apt-get update && apt-get install -qq -y nginx
	RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

	#Change ownership and permissions
	RUN chown -R www-data:www-data /var/lib/nginx
	RUN chmod 775 -R /var/log/nginx

	#Install the latest postgresql lib for pg gem
	RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" > /etc/apt/sources.list.d/pgdg.list && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --force-yes libpq-dev

	#Application home directory
	ENV APP /rails_app

	#Path for gems bundle install.
	ENV BUNDLE_PATH /box
	WORKDIR $APP

	#Add default nginx config
	ADD config/container/nginx-sites.conf /etc/nginx/sites-enabled/default

	#Add default unicorn config
	ADD config/unicorn.rb $APP/config/unicorn.rb

	#A script to start off unicorn and nginx and give executable permissions to it.
	ADD config/container/start-server.sh /usr/bin/start-server
	RUN chmod +x /usr/bin/start-server

	#Install Rails App in the container
	ADD . $APP

	ENV RAILS_ENV development

	#Expose port 80 for nginx
	EXPOSE 80

____

###### Configure unicorn

Add config/*unicorn.rb* for unicorn configuration

{% highlight ruby %}

# set path to application
app_dir = "/rails_app"
shared_dir = "#{app_dir}/tmp"
working_directory app_dir


# Set unicorn options
worker_processes 1
timeout 30

# Set up socket location
listen "#{shared_dir}/sockets/unicorn.sock", :backlog => 64

# Logging
stderr_path "#{app_dir}/log/unicorn.stderr.log"
stdout_path "#{app_dir}/log/unicorn.stdout.log"

# Set master PID location
pid "#{shared_dir}/pids/unicorn.pid"

{% endhighlight %}

____

###### Configure nginx

We add the configuration files in a container folder, and the *nginx-sites.conf* file is as follows:

	upstream unicorn_server {
	  server unix:/rails_app/tmp/sockets/unicorn.sock fail_timeout=0;
	}

	server {
	  listen 80;

	  root /rails_app/public;
	  try_files $uri @unicorn_server;

	  location @unicorn_server {
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header Host $http_host;
	    proxy_set_header X-Forwarded-Proto https; 
	    proxy_redirect off;
	    proxy_pass http://unicorn_server;
	  }

	  location ~ ^/(assets|images|javascripts|stylesheets|swfs|system)/ {
	    gzip_static on;
	    expires max;
	    add_header Cache-Control public;
	    add_header Last-Modified "";
	    add_header ETag "";

	    open_file_cache max=1000 inactive=500s;
	    open_file_cache_valid 600s;
	    open_file_cache_errors on;
	    break;
	  }
	}

____

###### Docker-compose to get the app running

Once nginx and unicorn are configured, to get our app up and running we use a small shell script to start all the services (config/container/start-server.sh) which is the step 13 in our dockerfile.

The shell script is as follows 

	#!/bin/bash

	truncate -s 0 $APP/tmp/pids/unicorn.pid
	bundle check || bundle install
	bundle exec rake assets:precompile
	bundle exec unicorn -c config/unicorn.rb -D
	service nginx restart


*Line 1*: Since we will be mounting a data-volume to the container, if unicorn is started once in a container its pid will be saved. To restart it in another container we first clear the old pid and start a new unicorn master process.

*Line 2*: Then we check if all the gems are installed in the app, if not run bundle install. This is the reason we dont include 'bundle install' commmand in the dockerfile. Only the first time we run bundle install in the gem box (which is a busy box tiny image approx. 2.5 mb which acts as a gem cache).

Once the gem dependencies are satisfied, unicorn is started in daemonized mode and nginx is started as well.

We can now have our *docker-compose.yml* which allows us to easily link all the containers to get our app running. Add the following to docker-compose.yml in the application root directory.

{% highlight ruby %}

db:
  image: postgres
  ports:
    - "5432"

gembox:
  image: busybox
  volumes:
    - /box

app:
  build: .
  command: /usr/bin/start-server
  volumes:
    - .:/rails_app
  volumes_from:
    - gembox
  links:
    - db
  ports:
    - "100:80"

{% endhighlight %}

The above yml file creates 3 containers:

1. A db container 
	* built from official postgres image 
	* exposing port 5432.
2. A gembox container 
	* built from busybox image
	* has '/box' as shared volume which is the BUNDLE_PATH for bundle install.
3. Our app container
	* built from the dockerfile we wrote earlier
	* with the present working directory being mounted as a data-volume to the app inside the container.
	* volumes_from which indicates the volume shared by gembox is mounted to this container and hence acts as a gem cache.
	* this container is linked to the db container for the postgres database.
	* the port insode the container (80) is bound to port 100 on the localhost.
	* lastly, the command /usr/bin/start-server, which is our shell script, is executed.

Once, we have the docker-compose.yml file ready the whole application, along with the database can be up in one single command

```
$ docker-compose up
```

Also, for setting up the database in the container run

```
$ docker-compose run app rake db:setup
```

The first time you run the app, it checks for the gem file dependencies as we have mentioned in our script

<img  src="{{site.baseurl}}/images/docker/ruby_app/ruby-rails-nginx-unicorn/first-time-up.png" >



It can be seen that gems are installed into /box in gembox container

<img  src="{{site.baseurl}}/images/docker/ruby_app/ruby-rails-nginx-unicorn/installed-in-box.png" >



To check if your gems are cached, try restarting the app.

<img  src="{{site.baseurl}}/images/docker/ruby_app/ruby-rails-nginx-unicorn/gem-satisfied.png" >



Now, if you go to localhost:100 you should see your app up and running.

<img  src="{{site.baseurl}}/images/docker/ruby_app/ruby-rails-nginx-unicorn/rails-up.png" >

[dofi]: dockerfile.html
[d]: dockerfile.html
[here]: http://unicorn.bogomips.org/DESIGN.html