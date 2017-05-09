---
layout: post
title: "Winter Series - The(or a) Spring Way"
date: 2017-05-08T19:26:16-06:00
---

Getting back to the Spring and Xenon comparison, I've implemented pretty much the same participant management capability for my Race Management System using the Spring ecosystem: [Spring Boot](https://projects.spring.io/spring-boot/), [Spring Data MongoDB](http://projects.spring.io/spring-data-mongodb/) and [Spring Data REST](http://projects.spring.io/spring-data-rest/). You can find the Github repo [here](https://github.com/jrrickard/ws-spring). For this simple use case, we can provide pretty similar capabilities using these three Spring projects, although we get to the end result in a diffeerent way. The first difference is that I'm obviously using MongoDB for persistence instead of the native persistence that you can leverage with Xenon. This means that I need to run Mongo plus my service. I'll use Docker for that. In a later post, I'll get into comparing how to run multiple replicas of MongoDB or a multi-node group of Xenon here, but for now we'll just compare the developer use case with both.


```
$ docker run -p 27017:27017 -d mongo
f8cbf4712a228a240d17da97e0cc366807b88c215d447129d90a65481a648c30
```

Now that I have a database, I'll the service using [Spring Initializer](http://start.spring.io/), which is super simple! 

![Bootstrap the project]({{site.url}}/images/spring/spring_initializer.png) 

This ends up giving me a Gradle based project skeleton and I can start building the service. I'll start with the data model. I'm also going to be using [Lombok](https://projectlombok.org/) annotations to remove some boilerplate code. 

```java
package io.github.jrrickard.ws.model;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.util.Date;

@Document
@Data
public class Participant {

    enum Gender {
        MALE,
        FEMALE
    }

    @Id
    private String id;
    private String firstName;
    private String lastName;
    private Date dateOfBirth;
    private Gender gender;
    private String city;
    private String state;
}

``` 

Just like with the Xenon example, this is pretty simple. Next, I need to make a [Spring Data Repository](https://docs.spring.io/spring-data/data-commons/docs/1.6.1.RELEASE/reference/html/repositories.html) for the Participant objects.


```java
package io.github.jrrickard.ws.persistence;

import io.github.jrrickard.ws.model.Participant;
import org.springframework.data.repository.CrudRepository;

public interface ParticipantRepository extends CrudRepository<Participant, String> {
}

```

When using [Spring Data Rest](http://docs.spring.io/spring-data/rest/docs/2.6.3.RELEASE/reference/html/) and a CRUD repository, you get a similar base capability that you'd get with a default Xenon stateful service. Spring goes further though and provides a number of additional capabilities, such as a PATCH operation and the ability to expose pre-defined queries.  

Next, we need to create a class that will provide our service configuration.

```java
package io.github.jrrickard;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;

@SpringBootApplication
@EnableMongoRepositories
public class WsParticipantApplication {

	public static void main(String[] args) {
		SpringApplication.run(WsParticipantApplication.class, args);
	}
}
```

This class is pretty simple as well. The important parts here are the @SpringBootApplication and @EnableMongoRepositories annotations. This will enable Spring Boot and enable some auto configuration of the service based on the classpath defined in the gradle build

```
  compile('org.springframework.boot:spring-boot-starter-data-mongodb')
  compile('org.springframework.boot:spring-boot-starter-data-rest')
  compile('org.springframework.boot:spring-boot-starter-web')
```

Spring Boot will enable Spring Data MongoDB because of the @EnableMongoRepositories annotation and turn on Spring Data Rest because of the spring-boot-starter-data-rest dependency.

With this in place, I can build the project and end up with an executable JAR like with the Xenon project. This will use Tomcat instead of the embedded Netty you'd get with Xenon. I'll use Docker to run this as well. 

Now, I can run the service.

```
$ docker run -d -p 8111:8080 -e SPRING_DATA_MONGODB_URI=mongodb://192.168.86.53:27017 -e SPRING_DATA_MONGODB_DATABASE=participnts spring-participant-service
```

Using Postman, I can now interact with the service

![Initial repository state]({{site.url}}/images/spring/spring-data-rest-initial.png) 

Next, I'll use Postman to create a record

![Postman create]({{site.url}}/images/spring/spring-data-rest-post.png)

Thanks to the capabilities provided by Spring Data REST, I've now created a record in the database. I can retrieve it with a simple GET request

![New record]({{site.url}}/images/spring/spring-data-rest-get.png)

I can also modify it with a PATCH operation and fetch it again to get the changed state.

![Patch request]({{site.url}}/images/spring/spring-data-rest-patch.png)

![Updated Record]({{site.url}}/images/spring/spring-data-rest-get-after-patch.png)


Using Spring, I was able to get a really simple CRUD based service up and running in only a few minutes. Almost everything is provided for you by the framework in this case, just like with Xenon. 


Obviously this is a pretty contrived example, but it demonstrates this simple use case using both technologies. It's hard to do an apples to apples comparison of Spring and Xenon, because they strive to do different things and have very different opinons about how to do the things they have in common. For this example, Spring was a litle easier to get going with. Getting an executable JAR with a RESTful interface to do CRUD operations on data with Spring was a bit simpler because Spring does a little more for you out of the box. Xenon, on the other hand, resulted in a slightly simpler deployment because I didn't need to also run MongoDB. 

I think the more interesting comparisons will come when comparing what building stateless and event-driven services  look like with both of the frameworks. 
