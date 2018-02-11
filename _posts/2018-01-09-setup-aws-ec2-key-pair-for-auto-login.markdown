---
layout: post
title: "Setup AWS EC2 key pair for quick SSH login"
date: 2018-01-09 10:05:00 +0800
description: Setup AWS EC2 key pair for Automatic SSH.
img: aws_ec2.jpg 
tags: [AWS, EC2]
---

Setup the AWS EC2 key pairs to quickly login with a short one line terminal command.

#### Create the EC2 instance

- Launch an instance say `Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-10acfb73`.
- Create a new key pair and save it to your local.
- Associate and Elastic IP or static IP to your instance.
- Add the private key to `~/.ssh/config` file for automatic login
- Don't forget to set permission on the pem file given as `chmod 400 ec2-key-pair`.

{% highlight bash %}
# append this to the ~/.ssh/config` file
Host ec2-server
    HostName <ec2-ip-address>.ap-southeast-1.compute.amazonaws.com
    User ubuntu
    IdentityFile <directory-path>/ec2-key-pair.pem
{% endhighlight %}

- Login to your terminal very easily like this as root for Ubuntu.

{% highlight bash %}

ssh -t ec2-server "sudo -s"

{% endhighlight %}


