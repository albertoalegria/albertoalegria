---
layout: post
title:  "RESTful services with Spring Boot - Part 1"
date:   2017-09-12 23:30:00
categories: spring-boot
permalink: /:categories/:title/
---

This article is the first of a series that's intended to guide you through the Spring Boot basics. In this part we create our first Spring Boot project and our first REST controller.

## What is Spring Boot?
Spring Boot is a tool that helps us to create Spring based projects with _almost_ zero configuration, speeding up the development process, this makes it our best choice when creating new projects.

Its [primary goal][1] is:
* Provide a faster and accessible getting started experience for all Spring development.

* Be opinionated out of the box, but get out of the way quickly as requirements start to diverge from the defaults.

* Provide a range of non-functional features that are common to large classes of projects.

* Absolutely no code generation and no requirement for XML configuration.

Even though there is a lot more information about Spring Boot, it's time to jump directly into the action. In this case we are going to create a REST service. 

## Creating a new project
The first thing you might want to do when creating a new Spring Boot project is going to the [Spring Initializr][2] page. This site allows us to download either a Maven or Gradle project with all our dependencies included. How cool is that?

The site looks like this:

![spring1]({{site.url}}/assets/images/posts/20170912/init1.png)

The next step is to choose whether you need Maven or Gradle, if you are using Java, Kotlin or Groovy and the Spring Boot version. In this case, I prefer Gradle over Maven, I’m going to use Java and the current Spring Boot version.

In the Project Metadata fields you need to write the group id and the artifact, but, what are those? Well, in plain English, the group id is often composed as "com." + your name or company name (without spaces), and the artifact is the project name.

Now it's time to select the project dependencies. Our dependencies are:

#### Web
This dependency allows us to develop web-based projects. It contains TomCat as the default server.

#### DevTools
Allows us to improve the development time with tools like Property Defaults, Automatic Restart and LiveReload. [More info.][3]

#### Jpa
With this we can use JPA and Hibernate in our project.

#### H2
Allows us to use the H2 database. Only for test purposes. You can switch to MySQL or PostgreSQL later.

Now, the page should look like this:

![spring2]({{site.url}}/assets/images/posts/20170912/init2.png)

By clicking the _Switch to the full version_ message, you can get a brief description of each dependency.

When you are done, press the _Generate Project_ button. A zip file with the project name will be downloaded, move it to your preferred location and extract it.

The next step is to launch your IDE and open a new project. I'll be using IntelliJ. Navigate to the path where you extracted the zip file and select that folder, a new window should appear:

![import]({{site.url}}/assets/images/posts/20170912/autoimport.png)


Just select the option _Use auto-import_ and click next. So far, that's all the configuration that we need.

The IDE now looks like this:

![look]({{site.url}}/assets/images/posts/20170912/firstlook.png)


See that message at the bottom that says Refreshing Gradle project? When this process is over, we will be ready to code.

## File structure
Before we started, let's have a look at the file structure, shall we?

![structure]({{site.url}}/assets/images/posts/20170912/structure.png)

As you can see, the structure is pretty straightforward. We have two folders, _main_ and _test_.

The _main_ directory has two subdirectories. The first one, _java_, contains our code, it has a directory whose name is composed of our group id plus the artifact. The second one, _resources_, contains two directories and one file called _application.properties_. The first directory, _static_, usually contains _.js_ and _.css_ files, while _templates_ contains _.html_ files. These directories are useful when we are managing our views with Spring Boot, using server-side template engines such as Thymeleaf.

The only task of our server is to manage the REST requests, so, both of these directories will not be very useful. The _application.properties_ file will be useful later, though.


The _test_ directory is used for, well, tests. More of that coming soon.

## Controllers
First of all, what's a controller? A controller is a component that manages HTTP requests. These components are identified with the `@Controller` annotation. Since we are creating a REST service, we will use `@RestController`.

Let's open our main class, given our artifact name, our main class will be _RestApplication_:

{% highlight java %}
@SpringBootApplication
public class RestApplication {
  public static void main(String[] args) {
    SpringApplication.run(RestApplication.class, args);
  }
}
{% endhighlight %}

If you've been using Spring MVC, you should note that there are a couple annotations missed. Well, that’s because of the `@SpringBootApplication annotation`. The above code is equivalent to:

{% highlight java %}
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class RestApplication {
  public static void main(String[] args) {
    SpringApplication.run(RestApplication.class, args);
  }
}
{% endhighlight %}

Since this is our first controller, let's make it inside the main class.

The `@RestController` annotation should be placed before the class declaration. Like:

{% highlight java %}
@SpringBootApplication
@RestController
public class RestApplication {
  public static void main(String[] args) {
    SpringApplication.run(RestApplication.class, args);
  }
}
{% endhighlight %}

Now, it's time to create our first HTTP request. This can be done through the `@RequestMapping` annotation. This annotation needs two parameters, the `name` and the `RequestMethod`. `Name` is the path that our controller will be serving, like `/users` or `/items`, the `RequestMethod` can be `Get`, `Head`, `Post`, `Put`, `Patch`, `Delete`, `Options` or `Trace`. We will be using as name `hello` and as `RequestMethod` `Get`

Next, we need to create the method that will create the response for this controller. Since we are using REST requests, you can make a simple method that returns any object and the application should work, but let's do these in a proper way. We will be using the `ResponseEntity` object. This is a generic class that receives an object then returns it, in most cases, with an `HttpStatus` object. Here will be returning a `String` containing the `Hello, buddy` message and an `HttpStatus.OK`.

{% highlight java %}
@RequestMapping(name = "hello", method = RequestMethod.GET)
  public ResponseEntity<String> sayHello() {
    return new ResponseEntity<String>("Hello, buddy", HttpStatus.OK);
}
{% endhighlight %}

By default, Spring Boot uses the port 8080 to start the server, so in order to check if the message displays correctly go to the path `http://localhost:8080/hello`. I'm using [Postman][4], but you can use your default browser:

![hello]({{site.url}}/assets/images/posts/20170912/hello.png)

That's all!

So far we created our first Spring Boot project and our first controller. In the next article we are going to create our first entity and we will learn to store records in a database, as well as other kinds of request methods. 



[1]: https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-introducing-spring-boot.html

[2]: http://start.spring.io/

[3]: https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html

[4]: https://www.getpostman.com/