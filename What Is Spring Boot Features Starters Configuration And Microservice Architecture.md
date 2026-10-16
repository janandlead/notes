# What Is Spring Boot?

## 1. Overview

**Spring Boot** is a project built on top of the Spring Framework that helps developers create production-ready Java applications quickly.

The Spring Framework provides the programming model and infrastructure. Spring Boot adds sensible defaults, automatic configuration, starter dependencies, embedded servers, externalized configuration, and operational features so developers can spend more time implementing business requirements.

In simple words:

> Spring Framework gives you the building blocks. Spring Boot assembles the common building setup so you can start the application with much less configuration.

Spring Boot is not a replacement for Spring Framework. It uses Spring Framework internally and makes common Spring applications easier to create, configure, run, test, and deploy.

## 2. Spring Framework versus Spring Boot

| Spring Framework | Spring Boot |
|---|---|
| Provides the core programming model | Uses the Spring Framework and simplifies its setup |
| Gives detailed control over configuration | Provides sensible defaults and auto-configuration |
| May require developers to configure more infrastructure | Reduces repetitive configuration |
| Requires selecting and coordinating framework modules | Provides starter dependencies for common application types |
| Can run in many deployment styles | Commonly creates a self-contained executable JAR |
| Focuses on infrastructure abstractions | Focuses on quickly building and operating applications |

Spring Boot still allows manual configuration. It does not hide the framework permanently or prevent customization. Developers can override Boot's defaults whenever the application needs different behavior.

## 3. Why Spring Boot was created

Traditional Spring applications could require a large amount of setup:

- Selecting many individual dependencies.
- Configuring a web application and servlet container.
- Registering framework components.
- Defining database and transaction settings.
- Configuring a server for local development.
- Creating deployment packages manually.

Spring Boot addresses this setup burden with:

- Starter dependencies.
- Auto-configuration.
- Embedded web servers.
- A convention-over-configuration approach.
- Executable JAR files.
- Externalized configuration.
- Built-in support for testing and production monitoring.

## 4. Main features of Spring Boot

### 4.1 Starter dependencies

A starter is a curated dependency descriptor for a common application capability. Instead of adding many libraries one by one, you add one starter.

For a REST API, you commonly add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

The starter brings the Spring libraries needed for web development and normally includes an embedded server and JSON support through managed transitive dependencies.

### 4.2 Auto-configuration

Spring Boot examines the classpath, existing beans, and configuration properties, then configures common infrastructure automatically.

For example, if a web starter is present, Boot can configure:

- Spring MVC infrastructure.
- A `DispatcherServlet`.
- An embedded web server.
- HTTP message converters.
- JSON serialization support.

Auto-configuration is conditional. Boot generally configures something only when the required classes are available and the developer has not already supplied a replacement bean.

### 4.3 Embedded servers

Spring Boot can package an application with an embedded server. A web application can therefore run as a JAR:

```bash
java -jar employee-command-service.jar
```

The team does not need to install and configure a separate application server for normal local development. The application starts its server as part of the Java process.

### 4.4 Executable JAR packaging

The application, compiled classes, and required libraries can be packaged into an executable JAR. This simplifies:

- Local execution.
- Container images.
- Continuous integration pipelines.
- Deployment to virtual machines or cloud platforms.

### 4.5 Externalized configuration

Configuration can be kept outside the compiled Java code. Common locations include:

- `application.properties`
- `application.yml`
- Environment variables
- Command-line arguments
- Profile-specific files such as `application-dev.yml`

This lets the same build artifact run in development, testing, staging, and production with different database URLs, ports, credentials, and feature settings.

### 4.6 Production-ready features

Spring Boot Actuator provides endpoints and infrastructure for monitoring and managing applications. Typical uses include:

- Health checks.
- Application information.
- Metrics.
- Environment inspection with appropriate security.
- Readiness and liveness checks.
- Graceful shutdown support.

Actuator endpoints must be exposed and secured intentionally. Monitoring information should not automatically be made public.

### 4.7 Developer productivity

Spring Boot supports fast development through:

- Spring Initializr.
- Consistent project structure.
- Maven and Gradle plugins.
- DevTools for local development.
- Convention-based defaults.
- Test slices and integration-test support.

## 5. Spring Boot project structure

A typical Maven project generated by start.spring.io looks like this:

```text
employee-command-service/
├── src/
│   ├── main/
│   │   ├── java/com/example/employeecommandservice/
│   │   │   ├── EmployeeCommandServiceApplication.java
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── model/
│   │   └── resources/
│   │       ├── application.properties
│   │       └── static/
│   └── test/
│       └── java/com/example/employeecommandservice/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md
```

The package layout can be organized by technical layer or by business feature. For a larger application, feature-based packaging often keeps related classes together:

```text
employee/
├── EmployeeController.java
├── EmployeeService.java
├── EmployeeRepository.java
└── Employee.java
```

## 6. The main Spring Boot application class

```java
package com.example.employeecommandservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class EmployeeCommandServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EmployeeCommandServiceApplication.class, args);
    }
}
```

### What `@SpringBootApplication` does

`@SpringBootApplication` is a convenience annotation that combines three important behaviors:

1. `@SpringBootConfiguration` identifies the application configuration.
2. `@EnableAutoConfiguration` enables Spring Boot's conditional auto-configuration.
3. `@ComponentScan` searches for Spring components in the application package and its subpackages.

Keep this class near the root package so controllers, services, repositories, and configuration classes can be discovered automatically.

## 7. Spring Boot starter modules

Starters are not the same as every Spring Framework module. They are convenient dependency bundles for application capabilities.

| Starter | Usage |
|---|---|
| `spring-boot-starter` | Core Boot application support |
| `spring-boot-starter-web` | Servlet-based web applications and REST APIs |
| `spring-boot-starter-webflux` | Reactive web applications and non-blocking APIs |
| `spring-boot-starter-data-jpa` | JPA and relational database persistence |
| `spring-boot-starter-jdbc` | JDBC database access |
| `spring-boot-starter-data-mongodb` | MongoDB access through Spring Data MongoDB |
| `spring-boot-starter-data-redis` | Redis data access and integration |
| `spring-boot-starter-security` | Authentication and authorization support |
| `spring-boot-starter-validation` | Bean Validation integration |
| `spring-boot-starter-test` | Testing support, including Spring Test and common test libraries |
| `spring-boot-starter-actuator` | Monitoring, health, metrics, and management features |
| `spring-boot-starter-thymeleaf` | Server-side HTML rendering with Thymeleaf |
| `spring-boot-starter-cache` | Spring cache abstraction |
| `spring-boot-starter-batch` | Batch-processing applications |
| `spring-boot-starter-amqp` | AMQP messaging, commonly RabbitMQ |
| `spring-kafka` or the appropriate Kafka starter support | Kafka producers, consumers, and messaging integration |
| `spring-boot-starter-mail` | Email integration through Spring's mail support |
| `spring-boot-starter-quartz` | Quartz scheduling integration |

The exact starter names and recommendations can vary by Spring Boot generation. Check the official Spring Boot documentation and the generated build file for the selected version.

## 8. Manual configuration in Spring Boot

Spring Boot reduces configuration, but it does not remove it.

### 8.1 Define a bean manually

```java
package com.example.employeecommandservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EmployeeConfiguration {

    @Bean
    public EmployeeIdGenerator employeeIdGenerator() {
        return new EmployeeIdGenerator();
    }
}
```

Spring Boot uses the manually defined bean instead of creating a conflicting default bean when the relevant auto-configuration is conditional.

### 8.2 Override a server port

In `src/main/resources/application.properties`:

```properties
server.port=8081
```

### 8.3 Configure an application property

```properties
employee.company-name=Example Corporation
```

Read it with `@Value`:

```java
@Value("${employee.company-name}")
private String companyName;
```

For groups of related properties, `@ConfigurationProperties` is usually easier to maintain:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "employee")
public class EmployeeProperties {

    private String companyName;

    public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }
}
```

### 8.4 Profiles

Profiles allow environment-specific configuration:

```text
application.properties
application-dev.properties
application-test.properties
application-prod.properties
```

Activate a profile from the command line:

```bash
java -jar employee-command-service.jar --spring.profiles.active=dev
```

Do not store production passwords directly in source control. Use environment variables, a secret manager, or the deployment platform's configuration mechanism.

## 9. Build a small employee REST service

### 9.1 Controller

```java
package com.example.employeecommandservice.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    @GetMapping("/message")
    public String message() {
        return "Employee command service is running";
    }
}
```

### 9.2 Service

```java
package com.example.employeecommandservice.service;

import org.springframework.stereotype.Service;

@Service
public class EmployeeService {

    public String serviceMessage() {
        return "Employee service is ready";
    }
}
```

### 9.3 Request flow

```text
GET /api/employees/message
        |
        v
EmployeeController
        |
        v
EmployeeService
        |
        v
HTTP response
```

In a real application, the service would call a repository, validate input, apply business rules, and return a response DTO rather than exposing database entities directly.

## 10. Database integration example

For a relational database application, add the appropriate Spring Data starter and database driver. A typical dependency set contains:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

Example configuration:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/employees
spring.datasource.username=employee_app
spring.datasource.password=${EMPLOYEE_DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=validate
```

The application still depends on PostgreSQL and its driver. Spring Boot configures the connection and JPA infrastructure from the dependencies and properties; it does not create the database server.

## 11. Running a Spring Boot application

### Run from an IDE

Open the class containing the `main` method and run it as a Java application.

### Run with Maven wrapper on Windows

```powershell
.\mvnw.cmd spring-boot:run
```

### Run with Maven wrapper on macOS/Linux

```bash
./mvnw spring-boot:run
```

### Build and run an executable JAR

```powershell
.\mvnw.cmd clean package
java -jar target/employee-command-service-0.0.1-SNAPSHOT.jar
```

### Test the endpoint

```bash
curl http://localhost:8080/api/employees/message
```

Expected response:

```text
Employee command service is running
```

## 12. Testing Spring Boot applications

### Application context test

```java
@SpringBootTest
class EmployeeCommandServiceApplicationTests {

    @Test
    void contextLoads() {
    }
}
```

Run tests with:

```powershell
.\mvnw.cmd test
```

### Web-layer test

For a controller-focused test, Spring Boot provides test-slice support such as `@WebMvcTest`. This loads the web layer without starting every application component.

### Integration test

Use `@SpringBootTest` when the test needs a larger part of the application context. For database or messaging integration tests, use test containers or an appropriate controlled test environment.

## 13. Spring Boot and microservice architecture

A **microservice architecture** divides a larger system into independently deployable services. Each service focuses on a business capability and communicates with other services through APIs or messages.

For an employee platform, the services might be:

```text
Employee Platform
├── employee-command-service
├── employee-query-service
├── department-service
├── payroll-service
├── leave-service
├── notification-service
└── reporting-service
```

Each service can be a separate Spring Boot application with its own controller, service layer, repository, configuration, tests, build pipeline, and deployment unit.

### Typical microservice request flow

```text
Client
   |
   v
API Gateway
   |
   +--> employee-command-service --> Employee Database
   |
   +--> employee-query-service   --> Read Database or Cache
   |
   +--> payroll-service          --> Payroll Database
   |
   +--> leave-service            --> Leave Database

Services can also communicate through an event broker:

employee-command-service -- EmployeeCreated event --> notification-service
```

### Characteristics of a Spring Boot microservice

- It has a focused business responsibility.
- It runs as an independent process.
- It can be packaged as an executable JAR or container image.
- It usually owns its data and controls its schema.
- It exposes REST, GraphQL, gRPC, or messaging interfaces.
- It has its own configuration, health checks, logs, metrics, and deployment pipeline.
- It can be scaled independently from other services.

### Advantages

- Independent deployment of business capabilities.
- Independent scaling of high-traffic services.
- Smaller codebases for individual teams.
- Technology or database choices can vary when there is a strong reason.
- A failure in one service can be isolated if timeouts, retries, and fallbacks are designed correctly.

### Challenges

- Network calls are slower and can fail.
- Distributed transactions are more difficult than local transactions.
- Data consistency may be eventual rather than immediate.
- Logging and debugging require correlation IDs and centralized observability.
- Deployments, configuration, security, and monitoring require more infrastructure.
- API and event contracts must be versioned and maintained.

Microservices should be split around business capabilities, not merely around technical layers. Creating a service for every class or database table usually increases complexity without providing useful independence.

### Spring projects commonly used with microservices

| Need | Spring technology |
|---|---|
| Service implementation | Spring Boot and Spring Framework |
| HTTP API | Spring MVC or Spring WebFlux |
| Database access | Spring Data JPA, JDBC, MongoDB, Redis, or another Spring Data module |
| Authentication and authorization | Spring Security |
| Service-to-service HTTP calls | Spring `RestClient`, `WebClient`, or Spring Cloud OpenFeign |
| API gateway | Spring Cloud Gateway |
| Centralized configuration | Spring Cloud Config |
| Service discovery | Spring Cloud discovery integrations when required |
| Resilience | Spring Cloud Circuit Breaker and compatible implementations |
| Messaging | Spring Kafka, Spring AMQP, or Spring Cloud Stream |
| Distributed tracing and metrics | Micrometer, Actuator, and an observability backend |
| Contract testing | Spring Cloud Contract |

The exact Spring Cloud components depend on the deployment platform and operational requirements. Do not add every distributed-systems component automatically.

## 14. House-construction analogy

Think of Spring Framework as the engineering principles, construction materials, and reusable building systems. Think of Spring Boot as a prepared construction site with common tools and default arrangements already in place.

| House construction | Spring Boot |
|---|---|
| Standard house plan | Convention-based project structure |
| Construction manager | Spring Boot startup and application context |
| Ready-to-use material package | Starter dependency |
| Preconnected utilities | Auto-configuration |
| House with built-in electrical and plumbing systems | Executable JAR with embedded server |
| Site settings | `application.properties` or `application.yml` |
| Inspection checklist | Automated tests and Actuator health checks |
| Different plan for different locations | Spring profiles |
| Several specialized buildings connected by roads and utilities | Microservice architecture |
| Specialized contractor | Database, broker, cloud, or external service |

![House construction compared with a layered Spring application](assets/spring-framework-house-construction-comparison.png)

The analogy has an important limit: Spring Boot prepares and coordinates the application structure, but it does not replace the external systems. A real house still needs water, electricity, materials, and a location. A real Spring Boot application still needs a Java runtime, data store, drivers, deployment environment, and other required infrastructure.

## 15. Does Spring Boot remove the need for other technologies?

No. Spring Boot reduces integration and configuration work; it does not eliminate all other technologies.

For example, a production employee application may use:

| Responsibility | Technology example |
|---|---|
| Runtime | Java/JDK |
| Web application | Spring Boot and embedded server |
| Database | PostgreSQL |
| Database access | Spring Data JPA and PostgreSQL driver |
| Authentication | Spring Security and an identity provider |
| Asynchronous events | Kafka or RabbitMQ |
| Build | Maven or Gradle |
| Deployment | Docker, Kubernetes, VM, or cloud service |
| Monitoring | Actuator, metrics backend, logs, and traces |

Spring Boot gives these technologies a consistent application environment and sensible defaults. It does not manufacture the external services or remove their operational requirements.

## 16. When Spring Boot is useful

Spring Boot is especially useful when:

- You are building REST APIs or web applications in Java.
- You need database transactions and persistence.
- You want standard project conventions.
- You need to integrate security, messaging, scheduling, or batch processing.
- You want to deploy an application as a self-contained JAR or container.
- You need independently deployable services with clear business ownership.
- Your team values a large ecosystem and consistent production practices.

## 17. Common mistakes

### Treating Spring Boot as a replacement for Java

Spring Boot is a Java application framework. Developers still need Java, a JDK, a build tool, and a runtime.

### Adding every starter

Add only the starters required by the application. Unnecessary dependencies increase startup time, attack surface, upgrade effort, and maintenance complexity.

### Relying on auto-configuration without understanding it

Auto-configuration is convenient, but developers should know which beans and properties are active. Use the condition evaluation report and documentation when behavior is unexpected.

### Exposing Actuator endpoints without security

Management endpoints may reveal operational information. Expose only what is required and protect sensitive endpoints.

### Putting business logic in controllers

Controllers should handle HTTP concerns. Put business rules in services and persistence operations in repositories.

### Committing passwords to configuration files

Use environment variables or a secrets-management solution for credentials.

## 18. Summary

- Spring Boot is built on top of the Spring Framework.
- It makes Spring applications faster to create and easier to run.
- Starters provide curated dependency groups.
- Auto-configuration applies sensible defaults based on the classpath and application settings.
- Embedded servers allow applications to run as executable JARs.
- Profiles and externalized configuration support multiple environments.
- Actuator adds health, monitoring, and management capabilities.
- Spring Boot is a strong foundation for independently deployable microservices.
- Microservices introduce operational and communication complexity, so service boundaries should be based on real business and scaling needs.
- Spring Boot reduces integration work but still depends on Java, databases, drivers, build tools, deployment platforms, and other infrastructure.

## 19. Official references

- [Spring Boot project](https://spring.io/projects/spring-boot)
- [Spring Boot reference documentation](https://docs.spring.io/spring-boot/reference/)
- [Spring Boot system requirements](https://docs.spring.io/spring-boot/system-requirements.html)
- [Spring Initializr](https://start.spring.io/)
- [Spring Framework project](https://spring.io/projects/spring-framework)
