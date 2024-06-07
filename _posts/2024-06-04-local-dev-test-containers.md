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

For this tutorial, we will try to create a very simple sample application and leverage most of the information based on the [official Spring Boot documentation](https://docs.spring.io/spring-boot/reference/features/dev-services.html#features.dev-services.testcontainers).
The sample application will be a Spring Boot application that connects to a PostgreSQL database. By using test containers, we shouldn't need to manually create a database in our local computers and will instead have our app running against a containerized PostgreSQL instance.

## Sample application

We'll use the [Spring Initializr](https://start.spring.io/) to create a new Spring Boot project. We'll add the following dependencies:

- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Test Containers
- Lombok
- Spring Boot DevTools

The below code is for demonstration purposes only and not a complete application and whatever happens on it shouldn't affect the main point of this post.

```java
@RequestMapping("api/v1/register")
@RestController
@RequiredArgsConstructor
public class RegistrationController {

  private final RegistrationServiceImpl registrationService;

  @PostMapping("/company")
  public ResponseEntity<Object> registerCompany(@Valid @RequestBody Company company) {
    return new ResponseEntity<>(registrationService.registerCompany(company), HttpStatus.CREATED);
  }
}
```

```java
@Service
@RequiredArgsConstructor
public class RegistrationServiceImpl implements RegistrationService {

  private final CompanyRegistrationRepository companyRegistrationRepository;

  @Override
  public Company registerCompany(Company company) {
    CompanyEntity companyEntity = company.mapToEntity();

    companyEntity = companyRegistrationRepository.save(companyEntity);

    return companyEntity.mapToModel();
  }
}
```

```java
public interface CompanyRegistrationRepository extends JpaRepository<CompanyEntity, UUID> {
}
```

```java
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Data
public class Company {
  private UUID id;

  @NotBlank
  private String name;

  private String cnpj;
  private String address;
  private String phone;
  private int motoSpots;
  private int carSpots;

  public CompanyEntity mapToEntity() {
    return CompanyEntity.builder()
      .id(this.id)
      .name(this.name)
      .cnpj(this.cnpj)
      .address(this.address)
      .phone(this.phone)
      .motoSpots(this.motoSpots)
      .carSpots(this.carSpots)
      .build();
  }
}
```

```java
@Table(name = "company")
@Entity
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Data
public class CompanyEntity {
    @Id
    @GeneratedValue
    private UUID id;
    private String name;
    private String cnpj;
    private String address;
    private String phone;
    private int motoSpots;
    private int carSpots;

    public Company mapToModel() {
        return Company.builder()
                .id(this.id)
                .name(this.name)
                .cnpj(this.cnpj)
                .address(this.address)
                .phone(this.phone)
                .motoSpots(this.motoSpots)
                .carSpots(this.carSpots)
                .build();
    }
}
```

## Test Containers

Once our application is generated, we should see our main `Application.java` file inside the `src/main` folder. In addition, if we have a look at our **src/test** folder, aside from our `ApplicationTest` class, we should see a new class called *`TestApplication`* (or *Test* + *TheNameForYourApplicationClass*). *This class is the one that will be responsible for starting our PostgreSQL container.*

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

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

Now using the endpoint we just configured, we can test our application by hitting that endpoint. We'll do this, however, **not** by running Application inside the **main** folder,
but we'll **instead** run `TestApplication` inside the **test** folder. This will start our PostgreSQL container, create our database, and run our application against it.

PS: In case you get into an **error** where the **application fails to create the relations (tables)**, you may need to add the below line to your application.properties (or .yml) file:

```properties
spring.jpa.hibernate.ddl-auto=update
```

The below line will ensure that the tables are created automatically. You can also use `create` instead of `update` if you want to drop and recreate the tables every time the application starts.

And that's it.

Thank you for reading this post. I hope you found it useful.
