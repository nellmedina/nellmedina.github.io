---
layout: post
title: "Install Keycloak using Docker with Https"
date: 2018-01-15 22:55:00 +0800
description: Installing Keycloak with Docker.
img: keycloak-docker.png
tags: [Keycloak, Docker]
---

This will install keycloak using Docker Compose but with configurations to make it secure and under and Nginx proxy.

My version of Docker Compose in installing Keycloak is [here](https://github.com/nellmedina/keycloak-docker).


#### Create the volumes and run `docker-compose up`
{% highlight bash %}
docker volume create nginx_volume
docker volume create keycloak_volume
docker volume create keycloak_postgresql_volume
{% endhighlight %}

#### Inside the keycloak container, add a new socket-binding element to the socket-binding-group element of Keycloak `standalone.xml`
{% highlight bash %}
<socket-binding-group name="standard-sockets" default-interface="public"
    port-offset="${jboss.socket.binding.port-offset:0}">
    ...
    <socket-binding name="proxy-https" port="443"/>
    ...
</socket-binding-group>
{% endhighlight %}

#### Inside the postgres container, execute this SQL to allow https.
{% highlight bash %}
psql -d mydb -U myuser

update REALM set ssl_required='NONE' where id = 'master';
{% endhighlight %}

#### The `blacklabelops/nginx` creates the nginx configuration automatically which will need editing to make proxy work for keycloak.
{% highlight bash %}
# go to /etc/nginx/conf.d/server1.conf and add an upstream just above the server block
upstream keycloak.nellmedina.com {
     server keycloak:8080;
}

# go to /etc/nginx/conf.d/server1/reverseProxy.conf
location / {
          proxy_pass http://keycloak.nellmedina.com;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_set_header X-Forwarded-Proto https;
          proxy_set_header X-Forwarded-Port 443;
        }
{% endhighlight %}

#### Restart the nginx and keycloak server then access keycloak at https://keycloak.nellmedina.com.
