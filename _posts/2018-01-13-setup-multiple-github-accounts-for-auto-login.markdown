---
layout: post
title: "Setup Multiple Github accounts for quick SSH login"
date: 2018-01-13 12:01:00 +0800
description: Github configure Automatic SSH.
img: github-auto-ssh.png 
tags: [Github, SSH]
---

It's annoying to always having to enter username and password when committing changes.  I recommend cloning git repositories using SSH to have automatic logins using `ssh-agent` on your Mac.

There may be instance when you have 2 github accounts which are for personal and work. Here are some steps as guide.

Create the ssh keys for your personal and work accounts.
{% highlight bash %}
# start the ssh-agent in the background
eval `ssh-agent -s`

ssh-keygen -t rsa -b 4096 -C "personal@gmail.com"
ssh-keygen -t rsa -b 4096 -C "work@gmail.com"

# assuming these files created after ssh-keygen
~/.ssh/id_rsa_personal
~/.ssh/id_rsa_work

# add the public keys to your Github SSH settings
pbcopy < ~/.ssh/id_rsa_personal.pub
pbcopy < ~/.ssh/id_rsa_work.pub
{% endhighlight %}

Add your private keys to ssh-agent. Note: *This is not* <span style="color:red">*required*</span>.

{% highlight bash %}
ssh-add ~/.ssh/id_rsa_personal
ssh-add ~/.ssh/id_rsa_work

# to check private keys added
ssh-add -l

# to delete all private keys
ssh-add -D
{% endhighlight %}

To __skip__ adding private keys every shutdown, you may put your keys in `~/.ssh/config` file instead.
{% highlight bash %}
sudo nano ~/.ssh/config

# personal
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_personal
    IdentitiesOnly yes

# work
Host work.github.com
    HostName work.github.com
    User git
    IdentityFile ~/.ssh/id_rsa_work
    IdentitiesOnly yes
{% endhighlight %}

Change the `.git/config` of your cloned repository location.
{% highlight bash %}
# open config file
sudo nano .git/config

# since personal is using Host as github.com, then no need to change it here
[remote "origin"]
        url = git@github.com:personal/some_repo.git

# edit the domain name for work
[remote "origin"]
        url = git@work.github.com:work/some_repo.git
{% endhighlight %}

#### Test your automatic SSH logins.

{% highlight bash %}
# delete all ssh identities
ssh-agent -D

# go to your git location and do a pull and NO more authenticaiton errors
git pull
{% endhighlight %}