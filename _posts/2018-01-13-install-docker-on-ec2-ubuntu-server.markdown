---
layout: post
title: "Install Docker on EC2 Ubuntu server"
date: 2018-01-13 10:51:00 +0800
description: This shows installation steps of Docker and Docker Compose on your Ubuntu image.
img: docker-on-ubuntu.png 
tags: [AWS, Docker, Ubuntu]
---

This of Docker is like the Iphone in the DevOps world where you get to run your apps in small containers that can talk to each other. These are light-weight containers that you can easily build and destory in a snap.

This is a Dream come true for developers. No longer we say:

> It works on my machine.

#### Install Docker

First, add the GPG key for the official Docker repository to the system:
{% highlight bash %}
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

apt-cache policy docker-ce
{% endhighlight %}

Finally, install Docker:
{% highlight bash %}
sudo apt-get install -y docker-ce

# check that its running
sudo systemctl status docker

# by default Docker requires root privilege so add your user
sudo usermod -aG docker ${USER}
{% endhighlight %}

#### Install Docker Compose

Get the currrent release.
{% highlight bash %}
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# set permissions
sudo chmod +x /usr/local/bin/docker-compose

# check version
docker-compose --version
{% endhighlight %}
