---
layout: post
title:  "RESTful services with Spring Boot - Part 2"
date:   2017-09-19 23:30:00
categories: spring-boot
---

In the [previous post]({{ site.baseurl }} {% post_url 2017-09-12-spring-rest-part-1%}), we learned how to create our first Spring Boot project as well as our first controller.

So, I think that is time to create something a _little_ more elaborated. We're going to create a music classifier, where we're going to be able to add songs, albums, genres, etc. So, from now on, I will be working on another Spring Boot project called _music_, you can keep working in the one you created previously, though. Let's do this.

## 1 Entities

Entities are the core of our application. They represent the objects that we use, such as _Users_, _Articles_, etc.

An entity is a class that represents a table inside our database. Is created by adding the `@Entity` annotation before the class declaration. 

In this case, I'm going to create an entity that represents a music album. In order to maintain our code organized, I'm going to put the class inside a package called `entities`.

{% highlight java %}
package com.albertoalegria.music.entities;

import javax.persistence.Entity;

@Entity(name = "music_album")
public class Album {
  ///
}

{% endhighlight %}

> #### Tip
> You can't use some words for your entities names (group, user, etc). To avoid this I highly recommend adding a distinctive identifier before.

The `@Entity` annotation has a `name` as an optional parameter, this will be the entity and the table name. If this parameter is not present, this value will be the same as our class name.

Now we're going to define our `Album` characteristics:

{% highlight java %}
@Entity(name = "music_album")
public class Album {
  private Integer id;
  private String name;
  private String artist;
  private Date date;

  /* Getters & Setters */
}
{% endhighlight %}

You can use a `Long` for your `id` if you wish.

The next step it's to tell JPA how these properties will be represented into the database, to do it we are going to use the `@Id` and `@Column` annotations:

{% highlight java %}
@Entity(name = "music_album")
public class Album {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "album_id")
  private Integer id;

  @Column(name = "album_name")
  private String name;

  @Column(name = "album_artist")
  private String artist;

  @Column(name = "album_date")
  private Date date;
}
{% endhighlight %}

The `@GeneratedValue` annotation is used to define the way our ids are going to be created. You can get more info [here][1]. Again, the property `name` at `@Column` is optional.

### 1.1 Validation
How can you assure that your data is always going to be created in a proper way? I mean, I don't think that you want someone to create an Album with an empty name or an invalid date. To avoid these problems you need to use validators:

{% highlight java %}
@Entity(name = "music_album")
public class Album {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "album_id")
  private Integer id;

  @NotBlank
  @Size(min = 1, max = 100)
  @Column(name = "album_name")
  private String name;

  @NotBlank
  @Size(min = 1, max = 50)
  @Column(name = "album_artist")
  private String artist;

  @NotNull
  @Past
  @Column(name = "album_date")
  private Date date;
}
{% endhighlight %}

Using validators gives you the guarantee that your fields are always going to be valid.

`@NotNull` means that this field must be not null.

`@Size` is used to validate strings. It takes two values, both represent the minimum and maximum length of our field.

`@Past` helps us to validate dates. It ensures that the given date is a _past_ date. E.g. If today is September 14th, 2017 and your date is September 15th, 2017, this value will be wrong.

`@NotBlank` validates that the input value contains at least one non-blank character. Since you may be confused with `@NotNull` and `@NotBlank`, I made a little table to illustrate their differences. Also, I added `@NotEmpty`:

| String | @NotNull   |   @NotEmpty |   @NotBlank |
| :----: | :--------: | :---------: | :---------: |
| _null_ | Not OK     | Not OK      | Not OK      |
| _""_   | OK         | Not OK      | Not OK      |
| _" "_  | OK         | OK          | Not OK      |
| _"Hello!"_| OK         | OK          | OK          |


> #### Info
> `@NotBlank` and `@NotEmpty` are _Hibernate_ validators, this means that if you are using another ORM, it won't be possible to use them

If you want to learn about other validators, please visit [this][2] page.

## 2 Repositories
Now that you have your first entity, you may be asking, how I put all of this into a database? With a repository of course!

Repositories are the JPA way to store and retrieve records from our database. There are simple to create and easily customizable. The repository to our `Album` entities should look like this:

{% highlight java %}
package com.albertoalegria.music.repositories;

import com.albertoalegria.music.entities.Album;
import org.springframework.data.jpa.repository.JpaRepository;

public interface AlbumRepository extends JpaRepository<Album, Integer> {
}
{% endhighlight %}

As you'll see, a repository is an _interface_ that extends the _JpaRepository_ generic class. This class receives our entity (`Album`) and his `Id` type. Until now that's all, later I'm going to teach you how to make queries with this same class.

## 3 Request methods

What is a request method? Well, according to the [MDN docs][4], HTTP request methods, or HTTP verbs, are methods to indicate the desired action to be performed for a given resource:


| Method    | Description |
| :-------: | :---------- |
| _GET_     | Requests a representation of a specified resource |
| _POST_    | Is used to submit an entity to the specified resource |
| _PUT_     | Replaces all current representations of the target resource with the request payload |
| _DELETE_  | Deletes the specified resource |
| _HEAD_    | Asks for a response identical to that of a _GET_ request, but without the response body |
| _CONNECT_ | Establishes a tunnel to the server identified by the target resource |
| _OPTIONS_ | Is used to describe the communication options for the target resource |
| _TRACE_   | Performs a message loop-back test along the path to the target resource |
| _PATCH_   | Used to apply partial modifications to a resource |

But, how to use them with Spring Boot?

In the first tutorial we made something like this:

{% highlight java %}
@RequestMapping(name = "hello", method = RequestMethod.GET)
public ResponseEntity<String> sayHello() {
  return new ResponseEntity<String>("Hey, buddy", HttpStatus.OK);
}
{% endhighlight %}

This example uses a _GET_ request method, this means that only retrieves data from the server, in this case, a string. 

As you can see, the request method is taken from the _RequestMethod_ class. This class is an enum containing all the methods showed in the above table. In addition, the _GET_, _POST_, _PUT_, _DELETE_ and _PATCH_ has his own annotations, so instead of using:

{% highlight java %}
@RequestMapping(name = "path", method = RequestMethod.GET)
@RequestMapping(name = "path", method = RequestMethod.POST)
@RequestMapping(name = "path", method = RequestMethod.PUT)
@RequestMapping(name = "path", method = RequestMethod.DELETE)
@RequestMapping(name = "path", method = RequestMethod.PATCH)
{% endhighlight %}

You can use:

{% highlight java %}
@GetMapping("path")
@PostMapping("path")
@PutMapping("path")
@DeleteMapping("path")
@PatchMapping("path")
{% endhighlight %}


## 4 Implementing _GET_

Let's implement what we've learned so far. First, I'm going to create a class called _AlbumController_. Then, I'm going to implement three _GET_ methods, one for retrieve all of my albums, another to retrieve an album by his id and another to retrieve albums by artist name:

{% highlight java %}
package com.albertoalegria.music.controllers;

import com.albertoalegria.music.entities.Album;
import com.albertoalegria.music.repositories.AlbumRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class AlbumController {
  @Autowired
  private AlbumRepository albumRepository;

  @GetMapping("albums")
  public ResponseEntity<List<Album>> getAll() {
    return new ResponseEntity<List<Album>>(albumRepository.findAll(), HttpStatus.OK);
  }

  @GetMapping("albums") // URL: /albums?id=1
  public ResponseEntity<Album> getAlbum1(@RequestParam int id) {
    return new ResponseEntity<Album>(albumRepository.getOne(id), HttpStatus.OK);
  }

  @GetMapping("albums/{id}") //URL: /albums/1
  public ResponseEntity<Album> getAlbum2(@PathVariable int id) {
    return new ResponseEntity<Album>(albumRepository.getOne(id), HttpStatus.OK);
  }
}

{% endhighlight %}

> The last two methods are kinda the same, except for the way the URL parameters are represented. I prefer the second one, but you can use whatever you want

See that `@Autowired` annotation? That's the way we call our repositories inside our controllers. This annotation _injects_ the desired beans into our bean. Basically, beans are objects that Spring manages. [Here's][5] an awesome StackOverflow answer that explains this concept better.

The `AlbumRepository` methods that we are using are the ones extended from the `JpaRepository` class.


### 4.1 Jpa Queries

Remember when I said that we are going to have another method that returns albums by his artist name? In order to implement this let's add the following line to our `AlbumRepository` bean:

{% highlight java %}
List<Album> findByArtist(String artist);
{% endhighlight %}

Just by adding this simple line is like having a query like:

{% highlight mysql %}
SELECT * FROM music_albums WHERE name = "artist_name"
{% endhighlight %}

Additionally, you can wrote your own Jpa queries, just be careful to use the same variable names as in your entity definition. I highly recommend you to read [this][6] awesome article.

Here's the new `AlbumRepository` method in action:

{% highlight java %}
@GetMapping("albums/{artist}")
public ResponseEntity<List<Album>> getByArtist(@PathVariable String artist) {
  return new ResponseEntity<List<Album>>(albumRepository.findByArtist(artist), HttpStatus.OK);
}
{% endhighlight %}

Ok, let's back to our controller.  The next step is to try to manage cases when the desired resource is not present. In the case of _getAll_ and _getByArtist_, if there are no resources the result will be an empty list. So we only going to edit the _getAlbum_ method. Remember, I'm using a `@PathVariable`, but you also can use `@RequestParam`:

{% highlight java %}
@GetMapping("albums/{id}")
public ResponseEntity<Album> getAlbum(@PathVariable int id) {
  if (albumRepository.exists(id)) {
    return new ResponseEntity<Album>(albumRepository.getOne(id), HttpStatus.OK);
  }
  return new ResponseEntity<Album>(HttpStatus.UNPROCESSABLE_ENTITY);
}
{% endhighlight %}

Why do we do this? Just imagine what will happen if we try to retrieve a resource without checking if it exists. In that case, the result will look like this:

![error]({{site.url}}/assets/images/posts/20170918/error_500.png)

What does that message mean? Basically, a _500 Internal Server Error_ code means that your server it's confused and doesn't know what to do. The proper way to manage these situations is to return the appropriate message, so you or the people using your service have an idea of what they are doing wrong. I this case the most suitable code is the _422 Unprocessable Entity_.

Why don't return the good old _404 Not Found_? That's because this error means that the URL is malformed, which is not the case here. What about _400 Bad Request_? A bad request is given when the request couldn't be understood because of a malformed syntax. Thus the _422 Unprocessable Entity_ is our best choice. The URL is correct, the server understands the request and the syntax is appropriate, but still is unable to process our instructions, in this case, get an _Album_ by an unexisting id.

[Here's][7] more information about status codes.

So, now the message should look like this:

![error2]({{site.url}}/assets/images/posts/20170918/error_422.png)

Looks nice, but just returning an empty message and a status code is not ok. You need to tell what is wrong with the request, so the users might be able to fix it.

### 4.2 Custom exceptions

In this step we re going to make a message that tells us what is wrong with our request. To achieve this we need to create a custom class that holds our message information.


{% highlight java %}
package com.albertoalegria.music.utils.exceptions;

import com.albertoalegria.music.utils.enums.MessageType;

public class ExceptionMessage {
  private long timestamp;
  private MessageType type;
  private String status;
  private String message;

  public ExceptionMessage() {}

  private ExceptionMessage(Builder builder) {
    this.message = builder.message;
    this.timestamp = builder.timestamp;
    this.type = builder.type;
    this.status = builder.status;
  }

  ///Getters & Setters

  public static class Builder {
    private String message;
    private long timestamp;
    private MessageType type;
    private String status;

    public Builder setMessage(String message) {
      this.message = message;
      return this;
    }

    public Builder setTimestamp(long timestamp) {
      this.timestamp = timestamp;
      return this;
    }

    public Builder setType(MessageType type) {
      this.type = type;
      return this;
    }

    public Builder setStatus(String status) {
      this.status = status;
      return this;
    }

    public ExceptionMessage build() {
      return new ExceptionMessage(this);
    }
  }
}
{% endhighlight %}

> As you can see, I'm implementing a [Builder Pattern][8], so you can build your message as you want. For example, let's say that you don't need timestamps or don't need a message type, instead of using 4 different constructors, you only use a `Builder`.

The class structure is very simple. Each message has the option to include a timestamp, a description, an Http status and a message type.

The `MessageType` property is an enum containing the messages types available within our application:

{% highlight java %}
package com.albertoalegria.music.utils.enums;

import com.fasterxml.jackson.annotation.JsonValue;

public enum MessageType {
    SUCCESS ("Success"),
    INFO ("Info"),
    WARNING ("Warning"),
    ERROR ("Error");

    private String name;
    MessageType(String name) {
        this.name = name;
    }

    @JsonValue
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
{% endhighlight %}

The _@JsonValue_ annotation helps us to give a little format to this parameter within our message. Instead of displaying _SUCCESS_, it will display _Success_. This is pure aesthetics.

The next step is to create a custom exception that will be thrown when the album doesn't exist:

{% highlight java %}
package com.albertoalegria.music.utils.exceptions;

public class UnprocessableEntityException extends RuntimeException {
  public UnprocessableEntityException(String message) {
    super(message);
  }
}
{% endhighlight %}

The exception implementation is going to be through a `@ControllerAdvice`:

{% highlight java %}
package com.albertoalegria.music.utils.exceptions;

import com.albertoalegria.music.utils.enums.MessageType;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

@ControllerAdvice
public class CustomExceptionHandler {
  @ExceptionHandler(UnprocessableEntityException.class)
  @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
  public static ResponseEntity<ExceptionMessage> handleUnprocessableEntity(UnprocessableEntityException uee) {
    ExceptionMessage.Builder builder = new ExceptionMessage.Builder();

    builder.setMessage("The URL and the syntax are correct, I understand the request, but I still can't " +
        "process your instructions because " + uee.getMessage());
    builder.setStatus(HttpStatus.UNPROCESSABLE_ENTITY.value() + " " + HttpStatus.UNPROCESSABLE_ENTITY.getReasonPhrase());
    builder.setTimestamp(System.currentTimeMillis());
    builder.setType(MessageType.ERROR);

    return new ResponseEntity<ExceptionMessage>(builder.build(), HttpStatus.UNPROCESSABLE_ENTITY);
  }
}

{% endhighlight %}

`@ControllerAdvice` allows us to handle exceptions through all our application, not only the _AlbumController_ class. `@ExceptionHandler` _listens_ to an exception, whenever this exception is thrown, this method will be performed. `@ResponseStatus` maps the exception to this response status.

Why use custom exceptions? Just imagine that instead of

{% highlight java %}
@ExceptionHandler(UnprocessableEntityException.class)
{% endhighlight %}

We use

{% highlight java %}
@ExceptionHandler(RuntimeException.class)
{% endhighlight %}

**Everytime** a _RuntimeException_ occurs this method will be performed. Even if it's not thrown by one of our controllers.

Now that this is finished, the new error message should look like this:

![error2]({{site.url}}/assets/images/posts/20170918/error_422_new.png)


I think this is enough for this article. In the next, we are going to learn how to create albums using a _POST_ method.


[1]: http://docs.oracle.com/javaee/7/api/javax/persistence/GenerationType.html
[2]: http://docs.oracle.com/javaee/6/tutorial/doc/gircz.html
[3]: https://en.wikipedia.org/wiki/Representational_state_transfer
[4]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
[5]: https://stackoverflow.com/a/17193458/4750070
[6]: https://docs.spring.io/spring-data/jpa/docs/1.3.4.RELEASE/reference/html/jpa.repositories.html
[7]: http://www.restapitutorial.com/httpstatuscodes.html
[8]: https://en.wikipedia.org/wiki/Builder_pattern