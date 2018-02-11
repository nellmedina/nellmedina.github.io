---
layout: post
title: "Install Jenkins master and slave with Docker"
date: 2018-01-14 06:33:00 +0800
description: Installing Jenkins on AWS EC2 Ubuntu using Docker Compose.
img: jenkins-docker.jpg
tags: [Jenkins, Docker]
---

There are a lot of CI/CD tools like Bamboo of Atlassian which makes sense for me because I use Jira a lot. 

I decided to try Jenkins mainly because its **Free** and has a large community base supporting thousands of plugins.

My version of Docker Compose files in installing Jenkins is [here](https://github.com/nellmedina/jenkins-docker).


#### Create volumes for backup
{% highlight bash %}
docker volume create jenkinsdata
docker volume create jenkinslogs
{% endhighlight %}

#### Set environment variables for Master say `jenkins-master.env`
{% highlight bash %}
# Jenkins port for accepting swarm slave connections
JENKINS_SLAVEPORT=50000

# Jenkins startup parameters.
# See: https://wiki.jenkins-ci.org/display/JENKINS/Starting+and+Accessing+Jenkins
JENKINS_PARAMETERS=
# Jenkins Mail Setup
SMTP_USER_NAME=
SMTP_USER_PASS=
SMTP_HOST=
SMTP_PORT=
SMTP_REPLYTO_ADDRESS=
SMTP_USE_SSL=
SMTP_CHARSET=

# Jenkins log file. Not necessary, because Jenkins logs to Docker.
JENKINS_LOG_FILE=
docker volume create jenkinslogs
{% endhighlight %}

#### Set environment variables for Slave say `jenkins-slave.env`
{% highlight bash %}
# note to secure passwords!
SWARM_JENKINS_USER=admin
SWARM_JENKINS_PASSWORD=password
SWARM_VM_PARAMETERS=-Xmx512m -Xms256m
SWARM_CLIENT_EXECUTORS=4

# SWARM_CLIENT_LABELS=jdk8 java
# SWARM_CLIENT_PARAMETERS=-name 'Super-Build' -description 'Super Client'
# SWARM_MASTER_URL=http://localhost:8090/
{% endhighlight %}

#### Create this `docker-compose.yml` and do a `docker-compose up` to run it.
{% highlight bash %}
version: '2.1'

services:
  # Jenkins Master
  jenkins:
    image: blacklabelops/jenkins:alpine
    container_name: jenkins
    hostname: jenkins
    networks:
      - jenkinsnet
    ports:
     - "8090:8080"
    volumes:
      - jenkinsdata:/jenkins
      - jenkinslogs:/var/log
    env_file:
      - jenkins-master.env
    labels:
      com.blacklabelops.description: "Jenkins Continuous Integration System"
      com.blacklabelops.service: "jenkins-master"
    restart: unless-stopped
  # Jenkins Slave
  slave:
    image: blacklabelops/swarm-jdk8
    networks:
      - jenkinsnet
    env_file:
      - jenkins-slave.env
    labels:
      com.blacklabelops.description: "Jenkins Swarm JDK-8 Slave"
      com.blacklabelops.service: "slave"
      com.blacklabelops.applications: "java maven gradle"
    restart: unless-stopped
volumes:
  jenkinsdata:
    external: true
  jenkinslogs:
    external: true

networks:
  jenkinsnet:
    driver: bridge
{% endhighlight %}