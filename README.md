# Spring Cloud demo in Scala

## Introduction

This is a step-by-step demo of how to use [Spring Cloud](http://projects.spring.io/spring-cloud/) to build microservices.
The demo shows the basics of how to use Spring Cloud's configuration service and some of the [Netflix](https://github.com/Netflix) microservices components that are supported by Spring Cloud: Eureka, Ribbon, Feign and Hystrix.

This demo is not more than a basic introduction to give you an idea of how you can use Spring Cloud to build microservices.
There are many more features and components that aren't shown here.

The source code is written in Scala.

The git repository for this demo contains a number of steps, that are tagged with git tags.
After cloning this repository, use the command `./goto step1` from a git bash shell to start with the first step.

Scripts are provided to build and run the services, but you can also load the project in an IDE and setup the run configuration so that you can run the services from your IDE.

## Step 1: Whiteboard Service

In the first step we will create a simple REST webservice using [Spring Boot](http://projects.spring.io/spring-boot/).

Creating a REST webservice with Spring Boot is very easy.
You can use the [Spring Initializr](http://start.spring.io/) to generate a Spring Boot project with the dependencies that you need.

For this demo we will create a simple service that allows users to write notes on a whiteboard.
Our whiteboard service is a Spring Boot application with [Spring Data JPA](http://projects.spring.io/spring-data-jpa/), [Spring Data REST](http://projects.spring.io/spring-data-rest/) and [Spring HATEOAS](http://projects.spring.io/spring-hateoas/).

Since we want to use Scala for this project, we need to add some things to the `pom.xml`:

* a dependency on the Scala library (`org.scala-lang:scala-library`)
* the build helper plugin (`org.codehaus.mojo:build-helper-maven-plugin`), which is used to add the `src/main/scala` and `src/test/scala` directories as source directories to the project
* the Scala plugin (`net.alchim31.maven:scala-maven-plugin`)

The Spring Boot application consists of a class `WhiteboardServiceApplication` and a companion object.
The class is annotated with the `@SpringBootApplication` annotation, and the companion object contains what we'd normally put in the `main()` method of a Spring Boot application if this were Java.

There is also a JPA entity `Note` and a corresponding Spring Data JPA repository, `NoteRepository`.

JPA is very much based on the Java way of doing things, and therefore there are a few things that are ugly when you use Scala instead of Java.

First of all, class `Note` is not at all idiomatic Scala - we are really just writing Java code with Scala syntax.
If this would have been idiomatic Scala, we would have made this a one-line immutable case class:

    case class Note(id: Long, createdDateTime: Date, authorName: String, content: String)

Second, the type of the `id` member has to be `java.lang.Long` (which we've imported here as `JavaLong`).
(Making it a `scala.Long` leads to an error message about serialization later on).

Finally, we have to use the `@BeanProperty` annotation on the members to generate JavaBeans-compatible getter and setter methods, because that is what JPA expects.

The repository is simple.
In Java this would be an interface, in Scala it's a trait.

We can now run the application.
The embedded Tomcat that Spring Boot starts up by default listens on port 8080, so we can go to [http://localhost:8080](http://localhost:8080) and see that our REST webservice is running, with HATEOAS / HAL support.

Note that there's a file `data.sql` in the `src/main/resources` directory which inserts some test data into the database - this will be picked up and executed automatically at startup by Spring Boot.
We're using an embedded, in-memory database ([H2](http://h2database.com)) for this demo.

## Step 2: Configuration Service

When you have a system that consists of many microservices that run across many servers, you'll want to centralize configuration.
It would be very cumbersome to manage the configuration of each microservice instance on each node where it is deployed.
Spring Cloud has a configuration service that very nicely fits in with the configuration mechanism of Spring Boot and the Spring Framework in general.

### Creating the configuration service

What we will do in this step is create another microservice - the configuration service.
The other services, such as the whiteboard service that we created, will get their configuration from the configuration service instead of from their own local `application.properties` file.

We add a module `config-service` to the project, which is another very plain and simple Spring Boot application.
This is the first module that will actually use components from Spring Cloud.
In the `pom.xml` we have to import the Spring Cloud dependencies POM in the `dependencyManagement` section, and then we add a dependency to `org.springframework.cloud:spring-cloud-config-server`.

To enable the configuration server, we add the `@EnableConfigServer` annotation to the application class.

We'll also need to set a few properties in `application.properties`.

First, we'll set the server port to 8888, because configuration clients will by default search for the configuration service on `localhost:8888`.

The configuration service by default gets the configuration properties that it serves from files in a git repository.
We have to set `spring.cloud.config.server.git.uri` to the URI of the git repository that will contain the configuration files.

### Making the whiteboard service a client of the configuration service

Now were are going to make some modifications to the whiteboard service so that it gets its configuration from the configuration service instead of from its own `application.properties` file.

You'll see that this is very easy and that we don't need to modify the source code.
The configuration server will act as just another source of configuration properties via Spring's property source mechanism.

We add a dependency on `org.springframework.cloud:spring-cloud-starter-config` to the whiteboard service.
Spring Boot's auto-configuration mechanism will then automatically configure the application so that it calls the configuration service to get configuration properties.

We must rename the `application.properties` of the whiteboard service to `bootstrap.properties` and add a property `spring.application.name` there.
The file `bootstrap.properties` is picked up early in the configuration process, and the application name is what the configuration service uses to find the configuration for this service.

We set `spring.application.name` to `whiteboard-service`, and then we make sure that there is a file `whiteboard-service.properties` in the configuration file git repository.

We set `server.port` in `whiteboard-service.properties` to 9090, so that we can see that if we start the config service and the whiteboard service, everything works.

Now we start the configuration service and the whiteboard service.
You'll see that the whiteboard service now gets its configuration from the config service and that it's running on port 9090 instead of the default port 8080.

Enter `./goto step3` in a git bash shell to go to step 3.
