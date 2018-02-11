---
layout: post
title: Simplified Blogging on Github for FREE
date: 2018-01-06 13:32:20 +0800
img: github_jekyll.jpg
tags: [Jekyll, Github]
---

Its amazing how some people would take time to write about stuff that can help other people experiencing similar issues to resolve it quicker. One perfect example is `stackoverflow.com` shared by millions of developers. The knowledge sharing expecting nothing in return inspired me the most to write my own blog. An added bonus is it serves as reference for myself.

I started out with Wordpress and MySQL using Docker compose deployed at AWS EC2 using my own domain which was Cool because you can serve wordpress and database on separate containers. It was fun messing around with __Docker Compose__ because spinning up Linux images is Fast. Unfortunately, I had to turn it down because hosting cost got pretty expensive.

Then almost 2 years later, I stumbled into one very simple but readable and beautiful `Tech` site that got me impressed and decided to give it a Try! I found out it was using Jekyll as its blogging engine then hosted freely at __Github__ pages. 

Its cool because it uses `Markdown` syntax that compiles it automatically to html. It also has a nice pattern of making layouts and conditions into your pages. I think with that pattern, the style of Blogging is more natural to Developers.

The Jekyll is free and is officially supported by Github pages which is also Free so its the best option for me now!

### How to do it?

I went straight to **Jekyll** official documentation page [here](https://jekyllrb.com/docs/quickstart/) and quickly had it tested locally on my Mac.

{% highlight ruby %}
# Install Jekyll and Bundler 
gem install jekyll bundler

# Create a new Jekyll
jekyll new myblog

# Change new directory
cd myblog

# Build the site
bundle exec jekyll serve

# Go to http://localhost:4000
{% endhighlight %}


If running localhost to your browser worked, then you may push it to your Github repository for a quick test. Go to the `Settings` tab then scroll down to **Github Pages** section. Next is point the drop-down to your _master_ branch and wait about 3 seconds to give Github time to build and deploy your Github page.

![]({{site.baseurl}}/assets/img/github_page_2.png)

If you come from the old way of Blog deployment like **Wordpress** and comparing it to **Github Pages** then you will be blown away at how easy it is to setup Jekyll. Plus its Free and your site is always up.






