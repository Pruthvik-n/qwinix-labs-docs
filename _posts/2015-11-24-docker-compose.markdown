---
layout: post
title:  "Docker Compose"
date:   2015-11-23 12:53:31
categories: docker part-6
topic: docker
---


When we run containers linked to each other, using the native CLI becomes hard to use.For example, if we want a rails application in a container to be linked to a postgres service running in another container, the command would be :

docker run -d -P --name db postgres

docker run -d -P --link db:db rails_app

If there are just two containers, this may look simple, but what if there are a lot of containers, say 4 of them and you want to link them, the command could get ugly, so instead we can link and run containers using docker-compose with a single command.

For using docker-compose we need to create a YAML file called *docker-compose.yml*

app:
  build: .
  command:
    - "rails server"
  ports:
    - "8000:8000"
  links:
    - db
db:
  image: postgres
  ports:
    - "5432:5432"


