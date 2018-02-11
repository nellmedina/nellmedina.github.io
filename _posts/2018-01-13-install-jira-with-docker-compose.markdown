---
layout: post
title: "Install JIRA with Docker Compose"
date: 2018-01-13 17:52:00 +0800
description: Installing jira supported with secured letsencrypt on your domain.
img: jira-software.png 
tags: [Jira, Docker, Letsencrypt]
---

This is to install JIRA Software secured by Letsencrypt with Docker Compose. Doing this manually will take hours to setup. 

Many thanks to [Steffen Bleul](https://github.com/blacklabelops) super genius DevOps skills, I can just re-use his Docker compose files. I recommend to visit his page and put a **star** over him at Github.

My version of Docker Compose files in installing Jira with a secured Domain is available [here](https://github.com/nellmedina/jira-docker-letsencrypt).

#### Quick Docker commands
> This is useful when you just want to clear your machine of Docker images and containers for a better testing experience.

{% highlight bash %}
# remove all images
docker rmi $(docker images -a -q)

# stop then remove containers
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)

# remove dangling volumes
docker volume prune
{% endhighlight %}

I have an EC2 Ubuntu instance called `mario` server that will have 3-5 apps deployed to it managed by using **Nginx** as proxy engine.

#### Expected sub-domains on `mario` server:
- jira.nellmedina.com
- jenkins.nellmedina.com
- keycloak.nellmedina.com

### Step 1: Create the SSL certificates and challenges

{% highlight yml %}
# create the volume to store certficates
docker volume create letsencrypt_certificates

# generate the certificates
# note that LETSENCRYPT_DOMAIN1 will become the parent certificate for the other domains
docker run --rm \
    -p 80:80 \
    -p 443:443 \
    -v letsencrypt_certificates:/etc/letsencrypt \
    -e "LETSENCRYPT_EMAIL=dummy@example.com" \
    -e "LETSENCRYPT_DOMAIN1=mario.nellmedina.com" \
    -e "LETSENCRYPT_DOMAIN2=jira.nellmedina.com" \
    -e "LETSENCRYPT_DOMAIN3=keycloak.nellmedina.com" \
    -e "LETSENCRYPT_DOMAIN4=jenkins.nellmedina.com" \
    blacklabelops/letsencrypt install
    
# create the volume to store certficate challenges
docker volume create letsencrypt_challenges
{% endhighlight %}

### Step 2: Test the domain certficates if working
> Make sure that ports 80 and 443 are open for testing. If no errors, then you can access https://mario.nellmedina.com and https://jira.nellmedina.com proxied by Nginx.
{% highlight yml %}
version: '2'

services:
  nginx:
    image: blacklabelops/nginx
    container_name: nginx_blacklabel
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - letsencrypt_certificates:/etc/letsencrypt
      - letsencrypt_challenges:/var/www/letsencrypt
    environment:
      - 'NGINX_REDIRECT_PORT80=true'
      - 'SERVER1PROXY_APPLICATION=test1'
      - 'SERVER1SERVER_NAME=mario.nellmedina.com'
      - 'SERVER1REVERSE_PROXY_LOCATION1=/'
      - 'SERVER1REVERSE_PROXY_PASS1=http://test1'
      - 'SERVER1HTTPS_ENABLED=true'
      - 'SERVER1HTTP_ENABLED=true'
      - 'SERVER1LETSENCRYPT_CERTIFICATES=true'
      - 'SERVER1CERTIFICATE_FILE=/etc/letsencrypt/live/mario.nellmedina.com/fullchain.pem'
      - 'SERVER1CERTIFICATE_KEY=/etc/letsencrypt/live/mario.nellmedina.com/privkey.pem'
      - 'SERVER1CERTIFICATE_TRUSTED=/etc/letsencrypt/live/mario.nellmedina.com/fullchain.pem'
      - 'SERVER2PROXY_APPLICATION=test2'
      - 'SERVER2SERVER_NAME=jira.nellmedina.com'
      - 'SERVER2REVERSE_PROXY_LOCATION1=/'
      - 'SERVER2REVERSE_PROXY_PASS1=http://test2'
      - 'SERVER2HTTPS_ENABLED=true'
      - 'SERVER2HTTP_ENABLED=true'
      - 'SERVER2LETSENCRYPT_CERTIFICATES=true'
      - 'SERVER2CERTIFICATE_FILE=/etc/letsencrypt/live/mario.nellmedina.com/fullchain.pem'
      - 'SERVER2CERTIFICATE_KEY=/etc/letsencrypt/live/mario.nellmedina.com/privkey.pem'
      - 'SERVER2CERTIFICATE_TRUSTED=/etc/letsencrypt/live/mario.nellmedina.com/fullchain.pem'
  test1:
    image: nginx
    container_name: test1_container
    hostname: test1
    restart: unless-stopped
    volumes:
      - ./test1:/usr/share/nginx/html
  test2:
    image: nginx
    container_name: test2_container
    hostname: test2
    restart: unless-stopped
    volumes:
      - ./test2:/usr/share/nginx/html

volumes:
  letsencrypt_certificates:
    external: true
  letsencrypt_challenges:
    external: true
{% endhighlight %}

### Step 3: Remove testing by stopping and removing related containers

{% highlight yml %}
# stop then remove containers (this removes all!)
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
{% endhighlight %}

### Step 4: Set the default environment variables that will be used by Docker Compose

{% highlight yml %}
# put this on a file say `defaultHttps.env`
export JIRA_TIME_ZONE=Asia/Singapore
export JIRA_DOMAIN_NAME=jira.nellmedina.com
export JIRA_DOMAIN_NAME_PARENT=mario.nellmedina.com
export JIRA_PORT80_REDIRECT=true
export JIRA_HTTPS=true
export JIRA_HTTP=true
export JIRA_POSTGRES_VERSION=9.4.6
export JIRA_VERSION=7.6.2
export JIRA_DB_USERNAME=jiradb
export JIRA_DB_PASSWORD=jiradb
export JIRA_PROXY_PORT=443
export JIRA_PROXY_SCHEME=https
export JIRA_DELAYED_START=10
export JIRA_JAVA_OPTIONS=" -Xms1g -Xmx2g"
export JIRA_NGINX_VERSION=1.10.1-r1

# update environment variables on your machine
source defaultHttps.env
{% endhighlight %}


### Step 5: Install JIRA!
> Notice that there is an independent container for Postgres database and have individual Volumes for easy backups
{% highlight yml %}
# need to create the volumes first
docker volume create nginx_volume
docker volume create jira_postgresql_volume
docker volume create jira_volume

# the docker-compose.yml file
version: '2'

services:
  nginx:
    image: blacklabelops/nginx
    container_name: nginx_blacklabel_prod
    ports:
      - '443:443'
      - '80:80'
    volumes:
      - nginx_volume:/home/nginx
      - letsencrypt_certificates:/etc/letsencrypt
      - letsencrypt_challenges:/var/www/letsencrypt
    environment:
      - 'TZ=${JIRA_TIME_ZONE}'
      - 'NGINX_REDIRECT_PORT80=${JIRA_PORT80_REDIRECT}'
      - 'SERVER1SERVER_NAME=${JIRA_DOMAIN_NAME}'
      - 'SERVER1REVERSE_PROXY_LOCATION1=/'
      - 'SERVER1REVERSE_PROXY_PASS1=http://jira:8080'
      - 'SERVER1HTTPS_ENABLED=${JIRA_HTTPS}'
      - 'SERVER1HTTP_ENABLED=${JIRA_HTTP}'
      - 'SERVER1LETSENCRYPT_CERTIFICATES=${JIRA_HTTPS}'
      - 'SERVER1CERTIFICATE_FILE=/etc/letsencrypt/live/${JIRA_DOMAIN_NAME_PARENT}/fullchain.pem'
      - 'SERVER1CERTIFICATE_KEY=/etc/letsencrypt/live/${JIRA_DOMAIN_NAME_PARENT}/privkey.pem'
      - 'SERVER1CERTIFICATE_TRUSTED=/etc/letsencrypt/live/${JIRA_DOMAIN_NAME_PARENT}/fullchain.pem'
      - 'SERVER1PROXY_APPLICATION=jira'
    restart: unless-stopped
  jira_postgresql:
    image: blacklabelops/postgres:${JIRA_POSTGRES_VERSION}
    container_name: jira_postgresql
    hostname: jira_postgresql
    volumes:
      - jira_postgresql_volume:/var/lib/postgresql/data
    environment:
      - 'TZ=${JIRA_TIME_ZONE}'
      - 'POSTGRES_DB=jiradb'
      - 'POSTGRES_USER=${JIRA_DB_USERNAME}'
      - 'POSTGRES_PASSWORD=${JIRA_DB_PASSWORD}'
      - 'POSTGRES_ENCODING=UNICODE'
      - 'POSTGRES_COLLATE=C'
      - 'POSTGRES_COLLATE_TYPE=C'
    restart: unless-stopped
  jira:
    image: blacklabelops/jira:${JIRA_VERSION}
    container_name: jira
    hostname: jira
    volumes:
      - jira_volume:/var/atlassian/jira
    environment:
      - 'TZ=${JIRA_TIME_ZONE}'
      - "CATALINA_OPTS=${JIRA_JAVA_OPTIONS}"
      - "JIRA_PROXY_NAME=${JIRA_DOMAIN_NAME}"
      - "JIRA_PROXY_PORT=${JIRA_PROXY_PORT}"
      - "JIRA_PROXY_SCHEME=${JIRA_PROXY_SCHEME}"
      - 'JIRA_DATABASE_URL=postgresql://${JIRA_DB_USERNAME}@jira_postgresql/jiradb'
      - 'JIRA_DB_PASSWORD=${JIRA_DB_PASSWORD}'
      - 'JIRA_DELAYED_START=${JIRA_DELAYED_START}'
    restart: unless-stopped

volumes:
  nginx_volume:
    external: true
  letsencrypt_certificates:
    external: true
  letsencrypt_challenges:
    external: true
  jira_postgresql_volume:
    external: true
  jira_volume:
    external: true
{% endhighlight %}

### Step 6: Setup auto-renew of SSL certificates
> This container will handshake with letsencrypt.org each month on the 15th and renew the certificate when successful.

{% highlight yml %}
# run it like this
docker-compose run -d letsencrypt

# the docker-compose.yml file
version: '2'

services:
  letsencrypt:
    image: blacklabelops/letsencrypt
    container_name: letsencrypt_autorenew
    volumes:
      - letsencrypt_certificates:/etc/letsencrypt
      - letsencrypt_challenges:/var/www/letsencrypt
    environment:
      - 'LETSENCRYPT_DOMAIN1=mario.nellmedina.com'
      - 'LETSENCRYPT_DOMAIN2=jira.nellmedina.com'
      - 'LETSENCRYPT_DOMAIN3=jenkins.nellmedina.com'
      - 'LETSENCRYPT_DOMAIN4=keycloak.nellmedina.com'

volumes:
  letsencrypt_certificates:
    external: true
  letsencrypt_challenges:
      external: true
{% endhighlight %}

### Step 7: Auto-reload the Nginx engine after certificates have been renewed
> Reloads Nginx configuration each month on the 15th over Docker without restarting Nginx! In order to achieve high availability!

{% highlight yml %}
version: '2'

services:
  jobber:
    image: blacklabelops/jobber:docker
    container_name: jobber_container
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - 'JOB_NAME1=ReloadNginx'
      - 'JOB_COMMAND1=docker exec nginx_blacklabel_prod nginx -s reload'
      - 'JOB_TIME1=0 0 2 15 * *'
      - 'JOB_ON_ERROR1=Continue'
{% endhighlight %}

