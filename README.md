Spring Boot RESTful Web Service Example
=======================================
The main purpose of this sample project is to demonstrate the capabilities of spring boot.
But additionally, I want to show challenging problems that can occur during the development
while using the Spring Boot. 

First goal is to show how it is easy to start a web service with embedded tomcat and embedded
H2 database. This is the main goal of the project.

Secondly, we are using Spring and I have used dependency injection. But what is a challenging
problem about dependency injection. Assume that you have two implementations ready for one
implementation, how are you going to select the implementation? I'll explain several ways but
also I'll demonstrate how we can select our implementation via external configuration so that
we can update our configuration and don't need to touch the code, restart our jar file and
that's all.

Thirdly, I also have demonstrated how to use Java Application Configuration within the double
implementation for the single interface scenario I've explained above.

Lastly, I will explain all the deployment details, the main configuration of the whole project
including H2 database configuration.

Moreover, I will also demonstrate how you are going to test your RESTful application with
Postman tool.

TOC
---
- [0 Prerequisite And Demo App](#0-prerequisite-and-demo-app) <br/>
- [1 About Spring Boot](#1-about-spring-boot) <br/>
- [2 Create Spring Boot Project With Maven](#2-create-spring-boot-project-with-maven) <br/>
- [3 Spring Boot Dependencies](#3-spring-boot-dependencies) <br/>
- [4 Making Uber Jar](#4-making-uber-jar) <br/>
- [5 Project Overview](#5-project-overview) <br/>
- [6 External Configuration Example](#6-external-configuration-example) <br/>
- [7 Application Properties](#7-application-properties) <br/>
- [8 H2 Database Preparation](#8-h2-database-preparation) <br/>
- [9 Sending And Receiving JSONs With Postman](#9-sending-and-receiving-jsons-with-postman) <br/>
  * [9-a- XXXXXXXX](#9-a-xxxxxxxx) <br/>
  * [9-b- XXXXXXXX](#9-b-xxxxxxxx) <br/>
  * [9-c- XXXXXXXX](#9-c-xxxxxxxx) <br/>
  * [9-d- XXXXXXXX](#9-d-xxxxxxxx) <br/>
  * [9-e- XXXXXXXX](#9-e-xxxxxxxx) <br/>
  * [9-f- XXXXXXXX](#9-f-xxxxxxxx) <br/>


 0 Prerequisite And Demo App
----------------------------
To use this project, you are going to need;

- Java JDK 8 (1.8)
- Maven compatibile with JDK 8
- Any Java IDE

We are going to build a demo app named as consultant-api. This will be a simple web service with
basic CRUD operations. I'm going to demonstrate default and external configuration, how to use
multiple implementation and autowire them within the code and outside the code with an external
configuration file. Our app will be a standalone application that we can use independently, and
we are going to use an embedded tomcat, an embedded H2 database.

[Go back to TOC](#toc)


 1 About Spring Boot
--------------------
Whenever there is a new framework on the town, you must think two thinks. One, why should I use this
framework which means "what are the benefits of this framework", also can be interpreted like "what
this framework solves?". Two, "When should I use this framework?", also can be interpreted as "on
which specific scenarios this framework is useful" or can be simplified as "what is the problem domain
of this framework?".

When we make a web service with spring framework, we have to generate a war file, we need to configure
web.xml, and also if we are going to use the connection pool, the configuration is costly. All of these
increases the cost of time. So instead of writing your code, doing your development, you a lot of time
is wasted during the configuration. This is where Spring Boot comes to the action. Spring Boot simplifies
configuration, reduces boilerplate code that puts no any value to your software development.

So, what Spring Boot solves is the time lost for the configuration. For example, you can create a web
service with Spring Boot that runs on an embedded Tomcat server which is automatically configured and
you don't have to deal with the configuration. You can do all your configuration parameters via default
application properties. Also you can connect to an H2 embedded database, same applies for the
configuration here. 

Secondly, you don't have to generate a war file. All Spring Boot applications run as a standalone java
jar file. Where is it useful then? If you are using a microservice architecture which runs especially
on a cloud (but not necessarily), then you can easily do your development via Spring Boot. In my opinion,
Spring Boot is one of the best frameworks you should use on such a scenario and architecture. You can
easily create simple web services, put them inside a Docker container (which is not a part of this
tutorial) and run them on the Amazon Web Services or on any cloud environment.

Additionally, you don't have to track the versioning of your dependencies. Normally, if you are using
a version of spring, then you have to use the appropriate versions of your other dependencies which are
dependent to the main dependency. With Spring Boot, you don't have to check out what version you have
to use for Jackson which is compatible with the version of Jersey. You don't even need to write the
version of your dependencies. I'm going to demonstrate all of these benefits.

[Go back to TOC](#toc)


 2 Create Spring Boot Project With Maven
----------------------------------------
What we need to setup a Spring Boot project. However there are other ways (like spring initializer),
I'll go with setting up our project with maven.

Because that we are creating a web application here, we will first create a maven project with web
application archetype, then we will add the spring boot dependencies;

You can use the following maven command to create a project. In this project, I've used this exact
maven command to create our project;

```
mvn archetype:generate -DgroupId=com.levent.consultantapi -DartifactId=consultant-api  -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

[Go back to TOC](#toc)


 3 Spring Boot Dependencies
---------------------------
How we make our project a Spring Boot project. We simply define a parent project in our POM file
just as below;

```
<!-- Spring Parent Project -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.3.1.RELEASE</version>
</parent>
```

Then our dependencies aware of our Spring Boot project. Then for example, if we are going to create
a web application which is true, then we add the Spring Boot Starter dependencies. On this project,
it is vital for us to add the dependency below;

```
<!-- Spring boot starter web: integrates and auto-configures  -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

See that we did not use the version attribute of the dependency block. That's what Spring Boot is 
going to handle but just adding this starter dependency, we will have all what we need like Jersey,
Jackson, and the rest. Also the versioning is managed by Spring Boot, we don't have to check if
the versions of our transitive dependencies are compatible with each other or not.

Moreover, in this project we will need an H2 embedded database. So we are going to add the following
dependency to our dependencies block;

```
<!-- H2 Embedded Database -->
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
</dependency>
```

The last thing additional to our dependencies in the POM file is to use the @SpringBootApplication
annotation on our [EntryPoint](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/EntryPoint.java) class.

With this, Spring Boot configuration is complete.

[Go back to TOC](#toc)


 4 Making Uber Jar
------------------

When we are developing a web service, let's say, using Jersey framework within the Spring context,
we make a .war file and upload it to a container. However, Spring Boot is a containerless framework.
So we do not need any web container, which means also we won't need to generate a .war file. What
we have to do is pack all the libraries and frameworks we are using in our project into a big jar
file, the Uber Jar (a.k.a. Fat Jar).

To do so, our build tool maven has a plugin named as Maven Shade Plugin. We are going to define
it within the build block of our POM file. You can see the sample build block as below;

```
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <transformers>
              <transformer 
implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.handlers</resource>
              </transformer>
              <transformer
implementation="org.springframework.boot.maven.PropertiesMergingResourceTransformer">
                <resource>META-INF/spring.factories</resource>
              </transformer>
              <transformer
implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.schemas</resource>
              </transformer>
              <transformer
implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.levent.consultantapi.EntryPoint</mainClass>
              </transformer>
            </transformers>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

As you can see, there are two plugins used in the plugins block of the build block above. I've
used maven compiler plugin so that I can define the source and destination version. The other
plugin is the Maven Shade Plugin, which we use in order to pack our Uber Jar.

In the Maven Shade Plugin block, we define our mainClass. Because our application is a standalone
Java application, we have to define the Entry Point, the starter class of our appliacation. The
name of this class is arbitrary.

You can check out the full POM file: [Project Object Model](https://github.com/bzdgn/simple-grizzly-standalone-restful-webservice-example/blob/master/pom.xml)

[Go back to TOC](#toc)


 5 Project Overview
-------------------

Our project consist of several layers. For the simplicity, if we exclude the [EntryPoint](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/EntryPoint.java) class 
which is located at the top level package, we have the following packages;

- [controller package](https://github.com/bzdgn/spring-boot-restful-web-service-example/tree/master/src/main/java/com/levent/consultantapi/controller)
- [service package](https://github.com/bzdgn/spring-boot-restful-web-service-example/tree/master/src/main/java/com/levent/consultantapi/service)
- [repository package](https://github.com/bzdgn/spring-boot-restful-web-service-example/tree/master/src/main/java/com/levent/consultantapi/repository)
- [model package](https://github.com/bzdgn/spring-boot-restful-web-service-example/tree/master/src/main/java/com/levent/consultantapi/model)

You can see the logical representation below of these packages;

![project-overview](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/ScreenShots/01_diagram.png)

Controller package has one controller class which is [ConsultantController](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/controller/ConsultantController.java). With [ConsultantController](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/controller/ConsultantController.java),
you can define all the RESTful methods. This class need to use a Service Layer implementation, thus
[ConsultantController](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/controller/ConsultantController.java) has a [ConsultantService](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/ConsultantService.java) interface and we are using the @Autowired annotation 
so that Spring context is going to find the appropriate implementation. In our case, we have only 
one Service implementation which is [ConsultantServiceImpl](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/consultant/impl/ConsultantServiceImpl.java).

You can also see the [InfoService](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/InfoService.java) interface wrapped inside the [ConsultantController](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/controller/ConsultantController.java) class. It has also
marked with the @Autowired annotation. I'll explain it later for the simplicity but our first focus
in this project overview is to explain the main structure of this simple application.

At service layer, we have the interface [ConsultantService](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/ConsultantService.java) and one implementation that fits with this
interface: [ConsultantServiceImpl](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/consultant/impl/ConsultantServiceImpl.java). You will see that [ConsultantServiceImpl](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/consultant/impl/ConsultantServiceImpl.java) has a [ConsultantRepository](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/repository/ConsultantRepository.java)
interface and that interface also marked with the @Autowired annotation.

At repository layer, we have a different situation. There are two implementation fits with this
[ConsultantRepository](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/repository/ConsultantRepository.java) interface;
- [ConsultantStubImpl](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/repository/impl/ConsultantStubImpl.java)
- [ConsultantJPARepositoryImpl](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/repository/impl/ConsultantJPARepositoryImpl.java)

So, how Spring handles if there are two implementation classes those implements one interface with
@Autowired annotation? How spring is going to select the implementation candidates? Normally, Spring
is going to get confused, throw Exceptions and you will find the following message inside the stack
trace;

```
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type [com.levent.consultantapi.repository.ConsultantRepository] is defined: expected single matching bean but found 2: consultantJPARepositoryImpl,consultantStubImpl
```

Because that there are two candidate implementation for one single interface, autowiring functionality
of Spring Context will fail. The solution is to use @Primary annotation. You can see it inside the
[ConsultantJPARepositoryImpl](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/repository/impl/ConsultantJPARepositoryImpl.java). When there are multiple implementation for the same interface marked with
@Autowired annotation, you have to use @Primary on one of the implementations.

But what if we want to have two different implementations and we want to change which implementation
to use, without changing the code ? Then we can use an external configuration, which I'm going to
explain it on next chapter.

[Go back to TOC](#toc)


 6 External Configuration Example
---------------------------------

Normally, Spring Boot configuration is defined with [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties). It is default location can be either under
src/main/resources folder, or under a subfolder of current directoy named with "config". I'm using the second way, created
a config folder under the project, and put the main [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties) file there. 

However, I also want to have the common properties on [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties) file, and in addition to it, for specific
purposes, I want to use a secondary properties folder. In this chapter, I'm going to demonstrate how to use externalized
custom properties file.

But first, let's go back to our previous problem that which I've talked about. The scenario is as follows: I've two or multiple
implementations for a single interface with @Autowired annotation, and I don't want to do any code change when I switch between 
the implementations. Spring's solution for that was using the @Primary annotation so that Spring Context is not going to throw an  exception because that it does not know which implementation to use.

Here is my solution;

- I create an externalized configuration file named as [implementation.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/implementation.properties)
- I've created [AppConfig](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/config/AppConfig.java) class to read the custom configuration file

If you look at to [AppConfig](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/config/AppConfig.java) class under the config package, you will see that the class file is marked with two annotations.
First it is marked with @Configuration annotation because it needs to be done before the injection of the autowired variable.
And secondly, it is marked with @PropertySource, so that we are going to point out which specific property file we are going
to use.

The process is as follows;

- Spring context searchs for the @Configuration annotation, finds the [AppConfig](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/config/AppConfig.java)
- Via [AppConfig](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/config/AppConfig.java), reads the [implementation.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/implementation.properties) file
- The greeter.implementation property's value is loaded to the impl variable in the [AppConfig](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/config/AppConfig.java)
- Via the #getImplementationFromPropertiesFile method, the implementation bean is created based on the config file
- The context has the right implementation based on the configuration
- During the creation of the [ConsultantController](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/controller/ConsultantController.java) class, Spring Context finds the autowired field: greeter which is an interface: [InfoService](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/service/InfoService.java)
- The implementation bean is autowired to this greeter field

You can see the overview of this process via the diagram below;

![process-overview](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/ScreenShots/02_diagram.png)

[Go back to TOC](#toc)


 7 Application Properties
-------------------------

Spring Boot solves our problem with automatic configuration as we use an embedded Tomcat and an embedded H2
database but how are we going to specify the running port of the Tomcat container, the target database, 
connection pool parameters and so on?

Spring Boot provides a default configuration properties file called as [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties). Within this file
there are hundreds of configuration parameters we can use. You can see the detailed parameter list via following
link;

[Spring Boot Application Properties Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

The default locations of the application.properties file is either somewhere within the classpath, for example
under src/main/resources in a maven project, or a inside config folder under current working directory. It is
better to put the file under config folder which will make it easy to deploy inside a docker container, but
the choice is yours. I place [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties) under config folder.

Because that we are using a server, H2 database, a datasource, a db connection pool and lastly, hibernate,
we should define parameters in this [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties) file, based on the [documented reference list](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

Let's take a detailed look;

- Server Configuration
&nbsp;&nbsp;&nbsp;&nbsp;The default configuration port is 8080, however we may want to change this. Thus I add the port configuration
&nbsp;&nbsp;&nbsp;&nbsp;as below;
```
#server port
server.port=8080
```
    
- H2 Database Configuration
&nbsp;&nbsp;&nbsp;&nbspWe also need to specify whether the console is activated, so that we can use H2 database via the console, create
&nbsp;&nbsp;&nbsp;&nbsp;our tables and initialize our db entries.
    
```
#H2 configuration
spring.h2.console.enabled=true
spring.h2.console.path=/h2
```

- DataSource Configuration
&nbsp;&nbsp;&nbsp;&nbsp;Instead of writing a connection string, we are defining the parameters via our properties file as below;
&nbsp;&nbsp;&nbsp;&nbsp;Notify that we defined our database as a file and the name of the database is "consultantapi". We are going to use it when
&nbsp;&nbsp;&nbsp;&nbsp;we need to connect the database via the console;

```
#Data source configuration
spring.datasource.url=jdbc:h2:file:~/consultantapi
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver
```

- Connection Pool Configuration
&nbsp;&nbsp;&nbsp;&nbsp; Here we define the connection pool parameters;

```
#DB Pool conf
spring.datasource.max-active=10
spring.datasource.max-idle=8
spring.datasource.max-wait=10000
spring.datasource.min-evictable-idle-time-millis=1000
spring.datasource.min-idle=8
spring.datasource.time-between-eviction-runs-millis=1
```

- Hibernate Configuration
&nbsp;&nbsp;&nbsp;&nbsp;We don't want Hibernate to delete our database entries on every restart of our server, so we need to configure as below;

```
#Hibernate Config
spring.jpa.hibernate.ddl-auto=false		#false for persistent database
```

You can check our project's application properties file via here: [application.properties](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/config/application.properties)

[Go back to TOC](#toc)


 8 H2 Database Preparation
--------------------------
Before using our database implementation instead of using our stub implementation, we need to prepare our table
and initial entries within the database. We have two things to do;

1. We need to create a CONSULTANT table to store our consultant model, defined in [Consultant](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/java/com/levent/consultantapi/model/Consultant.java) class.
2. We need to insert some trivial entries to the table.

The related SQL commands are located under the [consultant.sql](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/resources/consultant.sql)

As it is written on previous chapter, we have defined our H2 console with "/h2" postfix. Also we have defined our server
port as "8080", so connection url will be [http://localhost:8080/h2/](http://localhost:8080/h2/). We can connect our H2
database console as below;

![H2-console](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/ScreenShots/03_diagram.png)

After connecting to the database via the console, we can easily run our sql commands defined in [consultant.sql](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/src/main/resources/consultant.sql) as below;

![H2-console](https://github.com/bzdgn/spring-boot-restful-web-service-example/blob/master/ScreenShots/04_diagram.png)

Now our database is ready to go!

[Go back to TOC](#toc)


 9 9 Sending And Receiving JSONs With Postman
---------------------------------------------
