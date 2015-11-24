---
layout: post
title:  "AngulaJS Frontend"
date:   2015-09-17 11:05:32
categories: docker part-5
topic: docker
---

The below dockerfile is to containerize your AngularJs frontend and being served by an apche server.

We shall go through the commands one by one.

```
FROM ubuntu

# Install apache2,curl,ruby.
RUN apt-get update 
RUN apt-get install -y apache2 
RUN apt-get install -y curl git 
RUN apt-get install -y ruby ruby-dev build-essential

# Install nodejs and rubygems
RUN curl -sL https://deb.nodesource.com/setup | bash -
RUN apt-get install -y nodejs rubygems-integration

# Remove .deb packages
RUN apt-get clean

# Install npm,bower and gems sass,compass.
RUN npm install npm -g
RUN npm install -g bower
RUN npm install -g grunt-cli
RUN gem install --no-rdoc --no-ri sass
RUN gem install --no-rdoc --no-ri compass

RUN mkdir /var/www/html/angular_app
WORKDIR /var/www/html/angular_app

ADD . /var/www/html/angular_app

# Add your apache configuration.
ADD 000-default.conf /etc/apache2/sites-enabled/000-default.conf

RUN npm install
RUN bower --allow-root install -g

# Run grunt build which will build contents into the dist folder to be served by apache.
RUN grunt build

# Expose the default port for apache2.
EXPOSE 80

# Run apache2 in the foreground.
CMD ["apache2ctl","-D","FOREGROUND"]
```

A sample configuration is as follows, it can be configured the way you like it.

```
<VirtualHost *:80>

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/angular_app/dist

	ErrorLog /error.log
	CustomLog /access.log combined

</VirtualHost>

```


[dofi]: dockerfile.html
[d]: dockerfile.html
[here]: http://unicorn.bogomips.org/DESIGN.html