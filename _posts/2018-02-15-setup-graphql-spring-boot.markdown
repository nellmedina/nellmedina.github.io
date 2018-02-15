---
layout: post
title: "Setup GraphQL with Spring Boot"
date: 2018-02-15 18:37:00 +0800
description: GraphQL with Spring Boot.
img: graphql_img.png
tags: [GraphQL, Spring Boot]
---

The `REST` architecture is the most popular way to expose data through an `API`. But a better solution can be GraphQL which was open-sourced by Facebook on 2015 and released as a specification. GraphQL is a query language that offers an alternative model to developing APIs using a strong `type` system to provide a detailed description of an API.

A sample GraphQL project using Spring Boot is [here](https://github.com/nellmedina/graphql-spring-boot).

#### Java support to GraphQL specification using Spring Boot.

This is the main java library to start writing GraphQL in Java.

{% highlight xml %}
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java</artifactId>
    <version>6.0</version>
</dependency>
{% endhighlight %}

Creating code for GraphQL can be done in some ways like `Schema First` or `Code First`. I prefer using Schema First because its lesser code and much readable. The GraphQL Java Tools can parse your GraphQL schemas or types from a file instead of describing your types programmatically. This is similar technique used in graphql-tools in Javascript.

{% highlight xml %}
<dpendency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-tools</artifactId>
    <version>4.3.0</version>
</dependency>
{% endhighlight %}

 Instead of creating your REST controller, you can make use of GraphQL Servlet which implements a Servlet that supports both GET and POST requests for GraphQL queries.

{% highlight xml %}
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-servlet</artifactId>
    <version>4.7.0</version>
</dependency>
{% endhighlight %}

To make above dependencies work better for Spring boot, use below combination for your GraphQL dependencies.

{% highlight xml %}
<dependency>
	<groupId>com.graphql-java</groupId>
	<artifactId>graphql-java-tools</artifactId>
	<version>4.3.0</version>
</dependency>
<dependency>
	<groupId>com.graphql-java</groupId>
	<artifactId>graphql-spring-boot-starter</artifactId>
	<version>3.10.0</version>
</dependency>
<!-- to embed GraphiQL tool -->
<dependency>
	<groupId>com.graphql-java</groupId>
	<artifactId>graphiql-spring-boot-starter</artifactId>
	<version>3.10.0</version>
</dependency>
<!-- to embed GraphiQL tool -->
{% endhighlight %}

These Spring Boot dependencies support for GraphQL can be configured at `application.properties` or `application.yml`.

{% highlight yml %}
server:
    port: 9978
    # Must run on / otherwise graphiQL does not work currently
    context-path: /

graphiql:
    mapping: /graphql-client
    endpoint: /graphql-api
    enabled: true
    pageTitle: Blog GraphiQL

graphql:
    servlet:
       mapping: /graphql-api
       enabled: true
       corsEnabled: true
{% endhighlight %}

Using Postman or Curl for your queries can be very inconvenient so best practice is to make use of the built-in GraphQL client which was under `graphiql-spring-boot-starter`. This dependency provides GraphQL client in the browser by using http://localhost:9978/graphql-client. For Postman, curl or Apollo client can use http://localhost:9978/graphql-api.

#### Using Postman

![]({{site.baseurl}}/assets/img/graphql_postman.png)

#### Using Browser

![]({{site.baseurl}}/assets/img/graphql_browser.png)