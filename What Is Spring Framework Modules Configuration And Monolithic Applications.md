# What Is the Spring Framework?

## 1. Overview

The **Spring Framework** is an open-source Java framework for building enterprise applications. It provides reusable infrastructure for creating application objects, connecting components, handling web requests, accessing databases, managing transactions, processing messages, securing applications, and testing.

Spring's central idea is simple:

> The framework manages common technical work so developers can focus on business functionality.

Spring is not one single library. It is a modular ecosystem. The original Spring Framework provides the core programming model, while projects such as Spring Boot, Spring Data, Spring Security, Spring Batch, and Spring Cloud extend that model for particular application needs.

The official Spring Framework project describes its focus as providing the infrastructure or "plumbing" of enterprise applications while avoiding unnecessary ties to a particular deployment environment.

## 2. Why was Spring created?

Before Spring became popular, Java enterprise applications often required a large amount of infrastructure code and server-specific configuration. Developers had to manage objects manually, write repetitive setup code, handle transactions in a tightly coupled way, and place business logic inside classes that were difficult to test.

Spring addressed these problems with a lightweight programming model based on:

- Inversion of Control (IoC)
- Dependency Injection (DI)
- Plain Old Java Objects (POJOs)
- Declarative configuration
- Separation of business logic from infrastructure logic
- Consistent abstractions for web, data access, transactions, messaging, and testing

## 3. Important Spring concepts

### 3.1 Inversion of Control

In ordinary Java code, a class often creates the objects it needs:

```java
public class EmployeeService {

    private EmployeeRepository repository = new EmployeeRepository();
}
```

The service is now tightly coupled to a concrete repository. It controls object creation.

With Spring, the framework creates and supplies the dependency:

```java
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }
}
```

The control of object creation has moved from the application class to the Spring container. This is **Inversion of Control**.

### 3.2 Dependency Injection

Dependency Injection is the mechanism used to provide an object with the other objects it needs. The most recommended style is constructor injection because dependencies are explicit and the class can be immutable.

```java
@Service
public class PayrollService {

    private final EmployeeRepository employeeRepository;

    public PayrollService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }
}
```

Spring detects the repository bean, creates it, and passes it into the service constructor.

### 3.3 Spring Bean

A **bean** is an object managed by the Spring IoC container. A bean can be registered using annotations such as `@Component`, `@Service`, `@Repository`, or `@Controller`, or by defining it manually in a `@Configuration` class.

### 3.4 ApplicationContext

The `ApplicationContext` is Spring's container. It creates beans, injects dependencies, publishes events, loads resources, and coordinates application configuration.

## 4. Spring Framework modules and their uses

The Spring Framework is organized into functional module groups. You do not need every module in every application; select the modules that match your requirements.

### 4.1 Core Container

| Module | Main purpose | Typical usage |
|---|---|---|
| `spring-core` | Fundamental Spring utilities and the IoC foundation | Core framework behavior used by nearly all Spring applications |
| `spring-beans` | Bean factory and bean configuration | Creating and managing objects and their dependencies |
| `spring-context` | `ApplicationContext`, events, resources, internationalization | Application-wide configuration, events, resource loading, and bean management |
| `spring-expression` | Spring Expression Language (SpEL) | Expressions in configuration, annotations, and conditional logic |
| `spring-context-support` | Optional integrations for common enterprise services | Scheduling, caching, mail, and other context-level integrations |

The Core Container is the foundation of Spring. Dependency Injection, component scanning, bean scopes, lifecycle callbacks, and application events are built around this group.

### 4.2 AOP and instrumentation

| Module | Main purpose | Typical usage |
|---|---|---|
| `spring-aop` | Aspect-Oriented Programming support | Applying cross-cutting behavior such as logging, security checks, and transactions |
| `spring-aspects` | Integration with AspectJ | Using AspectJ-based aspects when Spring AOP alone is not sufficient |
| `spring-instrument` | Class instrumentation support | Application-server and class-loader instrumentation scenarios |

Cross-cutting concerns affect many classes. Logging, auditing, security, metrics, and transaction boundaries are examples. AOP lets these concerns be applied without duplicating the same code in every business method.

Example:

```java
@Transactional
public void transferSalary(Long sourceId, Long targetId, BigDecimal amount) {
    // Business logic executes inside a transaction.
}
```

The transaction behavior is applied around the method by Spring infrastructure.

### 4.3 Data Access and Integration

| Module | Main purpose | Typical usage |
|---|---|---|
| `spring-jdbc` | JDBC abstraction and exception translation | Database access without repeating low-level JDBC boilerplate |
| `spring-tx` | Programmatic and declarative transactions | Ensuring several database operations succeed or fail together |
| `spring-orm` | ORM integration | Integrating Hibernate, JPA, and other ORM technologies |
| `spring-r2dbc` | Reactive relational database access | Non-blocking database access in reactive applications |
| `spring-oxm` | Object/XML mapping abstraction | Converting Java objects to and from XML with supported mapper libraries |
| `spring-jms` | Java Message Service integration | Sending and receiving messages through JMS providers |
| `spring-messaging` | Messaging abstractions | Message-based applications, STOMP, WebSocket messaging, and integration endpoints |

Spring does not remove the need for a database driver, database server, JPA provider, or message broker. Instead, it provides consistent abstractions and integration so those technologies can be used with less application-specific code.

### 4.4 Web modules

| Module | Main purpose | Typical usage |
|---|---|---|
| `spring-web` | General web infrastructure | HTTP clients, web-aware application contexts, multipart uploads, and web utilities |
| `spring-webmvc` | Servlet-based Spring MVC | REST APIs, server-side web applications, controllers, request mapping, and form handling |
| `spring-webflux` | Reactive web framework | Non-blocking REST APIs and reactive web applications |
| WebSocket support | Full-duplex communication | Real-time notifications, chat, dashboards, and live updates |
| Web messaging support | STOMP and messaging over WebSocket | Structured messaging between browsers and servers |

A traditional Spring MVC request usually follows this path:

```text
Client request
    -> DispatcherServlet
    -> Controller
    -> Service
    -> Repository
    -> Database
    -> HTTP response
```

### 4.5 Testing modules

| Module or support | Main purpose | Typical usage |
|---|---|---|
| `spring-test` | Spring testing infrastructure | Loading an application context and testing Spring-managed components |
| Mock objects | Servlet and web mocks | Testing controllers without starting a real server |
| Spring MVC Test / MockMvc | MVC request testing | Testing routes, status codes, headers, and response bodies |
| WebTestClient | Web and reactive endpoint testing | Testing WebFlux applications and HTTP endpoints |
| TestContext Framework | Reusable test context management | Consistent setup for unit, integration, and slice tests |

Spring testing support works with testing libraries such as JUnit. Spring supplies integration and framework support; the testing library still executes the tests.

### 4.6 Other important Spring projects

These are related Spring projects, not all modules inside the core Spring Framework distribution:

| Spring project | Usage |
|---|---|
| Spring Boot | Opinionated setup, auto-configuration, starter dependencies, embedded servers, and production-ready application behavior |
| Spring Data | Consistent repositories and data-access abstractions for relational and NoSQL databases |
| Spring Security | Authentication, authorization, OAuth2, and protection against common web security problems |
| Spring Batch | Reliable processing of large volumes of records in jobs and steps |
| Spring Integration | Enterprise Integration Patterns and message-based system integration |
| Spring Cloud | Distributed-system patterns such as configuration, service discovery, gateways, and resilience |
| Spring for Apache Kafka | Spring abstractions for Kafka producers, consumers, and streams |
| Spring AMQP | Spring support for AMQP brokers such as RabbitMQ |
| Spring GraphQL | GraphQL server support for Spring applications |
| Spring Modulith | Structuring modular monoliths around business modules |
| Spring Session | Externalized and distributed HTTP session management |

Spring Boot is especially important for beginners: it uses the Spring Framework underneath but reduces the amount of manual setup required to start an application.

## 5. Manual configuration in Spring

### 5.1 What manual configuration means

Manual configuration means explicitly telling Spring how to create and connect application objects instead of relying entirely on component scanning and auto-configuration.

Manual configuration is useful when:

- A third-party class cannot be annotated with `@Component`.
- Multiple implementations of an interface exist.
- You need custom construction logic.
- You need different settings for different environments.
- You want precise control over bean names, scopes, or lifecycle.
- You are learning how the container works.

### 5.2 Java-based manual configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ApplicationConfig {

    @Bean
    public EmployeeRepository employeeRepository() {
        return new EmployeeRepository();
    }

    @Bean
    public EmployeeService employeeService(EmployeeRepository repository) {
        return new EmployeeService(repository);
    }
}
```

Spring reads the `@Configuration` class, calls the `@Bean` methods, registers the returned objects, and injects the repository into the service.

### 5.3 XML configuration

Older Spring applications may use XML configuration:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
         http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="employeeRepository"
          class="com.example.employee.EmployeeRepository" />

    <bean id="employeeService"
          class="com.example.employee.EmployeeService">
        <constructor-arg ref="employeeRepository" />
    </bean>
</beans>
```

XML is still supported in many existing systems, but Java configuration and annotations are more common in modern Spring applications.

### 5.4 Component scanning

Instead of declaring every bean manually, Spring can discover annotated classes:

```java
@Repository
public class EmployeeRepository {
}

@Service
public class EmployeeService {
}
```

Component scanning reduces configuration, but it is still based on the same container and dependency-injection model.

### 5.5 Manual configuration versus Spring Boot auto-configuration

| Manual Spring configuration | Spring Boot auto-configuration |
|---|---|
| Developer explicitly defines beans and infrastructure | Boot configures common infrastructure based on the classpath and properties |
| More control and visibility | Faster setup and fewer repetitive decisions |
| Useful for custom or legacy systems | Useful for most new applications |
| More configuration to maintain | Configuration can be overridden when required |

Spring Boot does not eliminate manual configuration. It provides sensible defaults, and developers can replace or customize those defaults.

## 6. The monolithic approach

### 6.1 What is a monolith?

A **monolithic application** is deployed as one application unit. Its modules may be separated internally, but the complete application is packaged and deployed together.

For example, an employee management system may contain these internal modules:

```text
Employee Management Monolith
├── Employee module
├── Department module
├── Payroll module
├── Leave module
├── Authentication module
└── Reporting module
```

All modules can run inside one Spring Boot process and may share one database, while still being separated into packages, services, repositories, and tests.

### 6.2 A modular monolith is not a disorganized monolith

A good monolith has boundaries:

- Each business area owns its classes and rules.
- Controllers do not contain database logic.
- Services contain business operations.
- Repositories handle persistence.
- Modules communicate through clear interfaces.
- Shared utilities are kept small and intentional.

Spring Modulith can help structure business modules inside a Spring Boot application, but it is optional. Normal package design and dependency rules are also useful.

### 6.3 Benefits of starting with a monolith

- One codebase to understand and deploy.
- Simple local development.
- Simple debugging and testing.
- Fewer network calls between business modules.
- Easier transactions across closely related operations.
- Lower infrastructure and operational cost.
- Faster development for a small or new team.

### 6.4 Limitations of a monolith

- A poorly structured monolith becomes difficult to change.
- One deployment may affect the entire application.
- Independent scaling of only one module is more difficult.
- A failure or memory problem can affect the complete process.
- Large teams may encounter more coordination and release conflicts.

Microservices are not automatically better. They introduce network communication, distributed transactions, deployment complexity, observability needs, and additional operational work. A well-designed modular monolith is often a strong starting point.

## 7. Can Spring build the entire application without other technologies?

Spring can provide most of the application-level structure, but the statement "no dependency on other technologies or libraries" needs an important clarification.

### What Spring provides

Spring can provide or coordinate:

- Object creation and dependency injection
- Web request handling
- REST APIs and MVC applications
- Transaction management
- Database access abstractions
- Validation integration
- Security
- Scheduling
- Caching
- Messaging integration
- Testing support
- Configuration and application events

### What Spring does not replace

An actual application still needs external infrastructure or supporting libraries, depending on its requirements:

| Requirement | Examples |
|---|---|
| Java runtime | JDK/JVM |
| Database | PostgreSQL, MySQL, Oracle, MongoDB, or another data store |
| Database driver | JDBC driver or reactive database driver |
| Build tool | Maven or Gradle |
| Web runtime | Embedded Tomcat, Jetty, or Netty, often supplied through Spring Boot |
| Test engine | JUnit and related test libraries |
| Message broker | Kafka, RabbitMQ, or another broker when asynchronous messaging is needed |
| Deployment platform | VM, container, Kubernetes, cloud platform, or physical server |
| Observability system | Metrics, logs, traces, and monitoring tools |

The accurate conclusion is:

> Spring reduces the amount of application code and configuration needed to integrate other technologies. It does not remove the need for every underlying technology.

This is one of Spring's greatest strengths: the application can use a PostgreSQL database today and change to another supported database with less business-code change because Spring provides common abstractions.

## 8. Real-time example: constructing a house

Imagine building a modern house.

### Without a framework

You personally coordinate every low-level activity:

1. Decide how to create every object and worker.
2. Manually connect electricity, water, doors, windows, and rooms.
3. Repeat safety checks in every room.
4. Handle every material detail yourself.
5. Change many unrelated parts when one supplier changes.

This is similar to Java classes creating their own dependencies and containing infrastructure code everywhere.

### With Spring Framework

Spring acts like a construction management system and a set of standard building services:

| House construction | Spring application |
|---|---|
| Building plan | Application architecture and configuration |
| Construction manager | Spring IoC container |
| Foundation | Core Container and dependency injection |
| Electrical wiring | Dependency connections between beans |
| Plumber and electrician | Infrastructure components such as data access and messaging |
| Rooms | Application modules such as employee, payroll, and leave |
| Doors and windows | Interfaces and extension points |
| Security guard | Spring Security |
| Water storage and flow control | Transaction and resource management |
| Building inspection | Automated tests and validation |
| House address | Application endpoint or service URL |
| Construction materials | Database, driver, web server, message broker, and other external technologies |

The construction manager does not manufacture every brick or replace the city water system. It coordinates the parts, applies standards, and makes the construction process easier to manage. Spring works in the same way: it coordinates application components and integrations while relying on Java, servers, databases, drivers, and other infrastructure where required.

![Comparison of house construction and a layered Spring application](assets/spring-framework-house-construction-comparison.png)

*The illustration compares the physical layers of a house with the logical layers of a Spring-based application. The analogy is conceptual: Spring organizes and connects application parts; it does not replace the external systems those parts use.*

## 9. Complete application example: employee management

Suppose a company needs an employee management application.

### Application requirements

- Add an employee.
- Search employees by department.
- Update employee details.
- Secure administrative operations.
- Store data in a relational database.
- Publish an event when an employee joins.
- Generate a monthly employee report.

### Spring-based design

```text
REST Controller
       |
Employee Service
       |
Employee Repository ---- PostgreSQL
       |
Transaction Management

Employee Service ---- Event Publisher ---- Kafka or RabbitMQ

Security Filter Chain ---- Authentication Provider

Scheduled Report Job ---- Report Service
```

Possible Spring technologies:

- Spring MVC for REST endpoints.
- Spring IoC and DI for connecting components.
- Spring Data JPA for repository access.
- Spring Transaction Management for atomic operations.
- Spring Security for authentication and authorization.
- Spring for Apache Kafka or Spring AMQP for events.
- Spring Batch or scheduled tasks for reports.
- Spring Test and MockMvc for automated tests.

The team can build the complete application as one modular monolith first. If the reporting or employee modules later need independent scaling, they can be separated carefully after the business boundaries are understood.

## 10. When should you choose Spring?

Spring is a strong choice when you need:

- A maintainable Java enterprise application.
- Clear separation between business logic and infrastructure.
- Dependency injection and testable components.
- REST APIs, web applications, or reactive services.
- Database transactions and integration with multiple data technologies.
- Authentication, authorization, messaging, scheduling, or batch processing.
- A solution that can start as a monolith and evolve later.

Spring may be more than you need for a tiny command-line program or a very small script. The best framework is the one that matches the application's requirements and the team's ability to operate it.

## 11. Summary

- Spring Framework is a modular Java framework for application infrastructure.
- Its foundation is the IoC container and Dependency Injection.
- Core modules support beans, context, expressions, AOP, web applications, data access, transactions, messaging, and testing.
- Manual configuration gives precise control using `@Configuration` and `@Bean`.
- Component scanning reduces repetitive configuration.
- Spring Boot uses Spring Framework and adds conventions and auto-configuration.
- A Spring application can be built as a modular monolith and split later when there is a clear reason.
- Spring reduces integration effort but does not eliminate databases, drivers, runtimes, build tools, or deployment infrastructure.
- The house-construction analogy helps explain Spring: it coordinates the structure and connections while external technologies provide the actual materials and services.

## 12. Official references

- [Spring Framework project overview](https://spring.io/projects/spring-framework)
- [Spring Framework reference documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Core technologies and IoC container](https://docs.spring.io/spring-framework/reference/core.html)
- [Spring testing support](https://docs.spring.io/spring-framework/reference/testing.html)
- [Spring projects](https://spring.io/projects)
