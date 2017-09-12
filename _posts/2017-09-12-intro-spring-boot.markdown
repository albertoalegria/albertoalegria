---
layout: post
title:  "RESTful services with Spring Boot"
date:   2017-09-12 13:00:00
categories: spring boot
---
# This article is a [WIP]

## What is Spring Boot?
Spring Boot is a tool that helps us to create Spring based projects with _almost_ zero configuration, speeding up the development process, this makes it our best choice when creating new projects.

Its [primary goal][1] is:
* Provide a faster and accessible getting started experience for all Spring development.

* Be opinionated out of the box, but get out of the way quickly as requirements start to diverge from the defaults.

* Provide a range of non-functional features that are common to large classes of projects.

* Absolutely no code generation and no requirement for XML configuration.

Even though there is a lot more information about Spring Boot, it's time to jump directly into the action. I this case we were creating a REST service. 

## Creating a new project
The first thing you might want to do when creating a new Spring Boot project is going to the [Spring Initializr][2] page. This site allows us to download either a Maven or Gradle project with all our dependencies included. How cool is that?

The site looks like this:

![spring1]({{site.url}}/assets/images/posts/20170912/init1.png)

The next step is to choose whether you need Maven or Gradle, if you are using Java, Kotlin or Groovy and the Spring Boot version. In this case, I prefer Gradle over Maven, I'd be using Java and the current Spring Boot version.

In the Project Metadata fields you need to write the group id and the artifact, but, what are those? Well, in plain english, the group id is often composed as "com." + your name or company name, without spaces, and the artifact is the projects name.

Now its time to select the project dependencies. The dependencies used in this case are: 
#### Web
This dependency allow us to develop web-based projects. It contains TomCat as default server.
#### DevTools
Allow us to improve the development time with tools like Property Defaults, Automatic Restart and LiveReload. [More info.][3]

#### Jpa



#### H2



[1]: https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-introducing-spring-boot.html

[2]: http://start.spring.io/

[3]: https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html