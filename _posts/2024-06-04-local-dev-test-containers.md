---
title: Local Development with Test Containers
date: 2024-06-04
categories:
- Development
tags:
- Spring Boot
- Test Containers
- Java
- Development
---

In this post, we will discuss how to use Test Containers for local development. Test Containers is a Java library that allows you to run Docker containers in your JUnit tests. This is useful for testing applications that rely on external services such as databases, message brokers, and other services, or tests such
as integration tests that require a running environment, e.g: Pact or Rest Assured tests against the localhost environment. However, the purpose of this tutorial is to show how to use Test Containers for local **development**, which is something that not many people are familiar with.

## Sample application

We will try to create a very simple sample application and leverage most of the information based on the [official Spring Boot documentation](https://docs.spring.io/spring-boot/reference/features/dev-services.html#features.dev-services.testcontainers). 
The sample application will be a Spring Boot application that connects to a PostgreSQL database. By using test containers, we shouldn't need to manually create a database in our local computers and will instead have our app running against a containerized PostgreSQL instance.

## Setting up the project

We'll do use the [Spring Initializr](https://start.spring.io/) to create a new Spring Boot project. We'll add the following dependencies:

- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Test Containers
- Lombok
- Spring Boot DevTools

Once our application is generated, we should see our main Application.java file inside the src/main folder. In addition, if we have a look at our **src/test** folder, aside from our ApplicationTest class, we should see a new class called *TestApplication* (or *Test* + *TheNameForYourApplicationClass*). *This class is the one that will be responsible for starting our PostgreSQL container.*

```java

@TestConfiguration(proxyBeanMethods = false)
public class TestApplication {

    @Bean
    @ServiceConnection
    PostgreSQLContainer<?> postgresContainer() {
        return new PostgreSQLContainer<>(DockerImageName.parse("postgres:latest"));
    }

    public static void main(String[] args) {
        SpringApplication.from(Application::main).with(TestApplication.class).run(args);
    }
}

```

Now assuming we have an endpoint configured within our application. We can run our application and test it by hitting the endpoint. We'll do this, however, **not** by running Application inside the **main** folder,
but we'll **instead** run `TestApplication` inside the **test** folder. This will start our PostgreSQL container, create our database, and run our application against it.

PS: In case you get into an issue where the application fails to create the relations (tables), you may need to add the below line to your application.properties (or .yml) file:

```properties
spring.jpa.hibernate.ddl-auto=update
```

The below line will ensure that the tables are created automatically. You can also use `create` instead of `update` if you want to drop and recreate the tables every time the application starts.

And that's it.

Thank you for reading this post. I hope you found it useful.
