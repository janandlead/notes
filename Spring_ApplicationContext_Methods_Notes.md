# Spring Boot ApplicationContext — Methods, Examples, and Interview Notes

## Learning objectives

By the end of this lesson, you should be able to explain what `ApplicationContext` does, retrieve and inspect Spring beans, read configuration, load resources, publish events, and decide when constructor injection is preferable to direct context access.

## 1. What is ApplicationContext?

`ApplicationContext` is Spring's central IoC (Inversion of Control) container interface. It extends `BeanFactory` capabilities and exposes application-level features such as message resolution, resource loading, application events, and environment access.

```java
import org.springframework.context.ApplicationContext;
```

In Spring Boot, `SpringApplication.run(...)` returns a `ConfigurableApplicationContext`, which is also an `ApplicationContext`.

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(DemoApplication.class, args);
        System.out.println(context.getId());
    }
}
```

> **Important:** Some methods below are inherited from parent interfaces (for example, `BeanFactory`, `ListableBeanFactory`, `MessageSource`, `ResourceLoader`, and `ApplicationEventPublisher`).

## 2. Quick reference

| Method | What it does |
|---|---|
| `getBean(String name)` | Gets a bean by name |
| `getBean(Class<T> type)` | Gets a bean by type (must be unambiguous) |
| `getBean(String name, Class<T> type)` | Gets a bean by name and type |
| `containsBean(String name)` | Checks whether a bean is available under a name |
| `containsBeanDefinition(String name)` | Checks whether a definition is registered locally |
| `getBeanDefinitionCount()` | Counts locally registered bean definitions |
| `getBeanDefinitionNames()` | Lists locally registered bean definition names |
| `getBeansOfType(Class<T> type)` | Returns matching beans as a map |
| `getBeanNamesForType(Class<?> type)` | Returns names matching a type |
| `isSingleton(String name)` | Checks singleton scope semantics |
| `isPrototype(String name)` | Checks prototype scope semantics |
| `getType(String name)` | Returns the bean's type, if determinable |
| `getAliases(String name)` | Returns aliases for a name |
| `getId()` | Returns context ID |
| `getApplicationName()` | Returns application name (may be empty) |
| `getDisplayName()` | Returns human-readable context name |
| `getParent()` | Returns parent context, if any |
| `getEnvironment()` | Provides configuration properties and active profiles |
| `getResource(String location)` | Resolves a resource |
| `getResources(String pattern)` | Resolves matching resources (when supported by resolver) |
| `getMessage(...)` | Resolves localized messages |
| `publishEvent(Object event)` | Publishes an application event |
| `getStartupDate()` | Returns context startup timestamp (epoch milliseconds) |
| `getAutowireCapableBeanFactory()` | Provides programmatic wiring capabilities |

## 3. Minimal example project

Use Spring Boot 3+, Java 21, and the `spring-boot-starter` dependency. Put classes under the same base package or its subpackages so component scanning can discover them.

**`EmployeeService.java`**

```java
package com.example.demo.service;

import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    public void display() {
        System.out.println("EmployeeService method executed");
    }
}
```

**`DemoApplication.java`**

```java
package com.example.demo;

import com.example.demo.service.EmployeeService;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(DemoApplication.class, args);
        EmployeeService service = context.getBean(EmployeeService.class);
        service.display();
    }
}
```

Expected application message:

```text
EmployeeService method executed
```

## 4. Bean lookup and inspection methods

### 4.1 `getBean()` — retrieve a managed object

```java
EmployeeService byType = context.getBean(EmployeeService.class);
EmployeeService byName = (EmployeeService) context.getBean("employeeService");
EmployeeService byBoth = context.getBean("employeeService", EmployeeService.class);
byType.display();
```

The default component name for `EmployeeService` is `employeeService`. If no bean matches, Spring throws `NoSuchBeanDefinitionException`; if multiple beans match a type without a unique candidate, it can throw `NoUniqueBeanDefinitionException`.

### 4.2 `containsBean()` — check bean availability

```java
System.out.println(context.containsBean("employeeService")); // true
```

### 4.3 `containsBeanDefinition()` — check a local definition

```java
System.out.println(context.containsBeanDefinition("employeeService")); // typically true
```

Unlike `containsBean`, this specifically checks locally registered bean definitions rather than resolving a bean through the hierarchy.

### 4.4 `getBeanDefinitionCount()` — count definitions

```java
System.out.println("Bean definitions: " + context.getBeanDefinitionCount());
```

Spring Boot also registers infrastructure and auto-configuration definitions; the count is usually much larger than the number of application-specific classes.

### 4.5 `getBeanDefinitionNames()` — list definitions

```java
for (String name : context.getBeanDefinitionNames()) {
    System.out.println(name);
}
```

### 4.6 `getBeansOfType()` — find every matching bean

```java
import java.util.Map;

Map<String, EmployeeService> services = context.getBeansOfType(EmployeeService.class);
services.forEach((name, service) -> System.out.println(name + " -> " + service));
```

### 4.7 `getBeanNamesForType()` — list matching names

```java
for (String name : context.getBeanNamesForType(EmployeeService.class)) {
    System.out.println(name);
}
```

Expected application bean name: `employeeService`.

### 4.8 `isSingleton()` — singleton scope

```java
System.out.println(context.isSingleton("employeeService")); // true by default
```

### 4.9 `isPrototype()` — prototype scope

```java
System.out.println(context.isPrototype("employeeService")); // false by default
```

To make the bean prototype-scoped:

```java
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

@Service
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class EmployeeService {
    public void display() {
        System.out.println("Prototype service");
    }
}
```

Then `context.isPrototype("employeeService")` is `true`.

### 4.10 `getType()` — inspect bean type

```java
Class<?> type = context.getType("employeeService");
System.out.println(type); // class com.example.demo.service.EmployeeService (typically)
```

### 4.11 `getAliases()` — retrieve aliases

```java
for (String alias : context.getAliases("employeeService")) {
    System.out.println(alias);
}
```

If no aliases exist, the returned array is empty.

## 5. Application context metadata

### 5.1 `getId()`

```java
System.out.println(context.getId()); // often "application"
```

### 5.2 `getApplicationName()`

```java
System.out.println(context.getApplicationName());
```

It may be an empty string in a standalone application; it is not necessarily the same as `spring.application.name`.

### 5.3 `getDisplayName()`

```java
System.out.println(context.getDisplayName());
```

The value depends on the concrete context implementation.

### 5.4 `getParent()`

```java
ApplicationContext parent = context.getParent();
System.out.println(parent); // often null in a standalone Boot app
```

### 5.5 `getStartupDate()`

```java
System.out.println(new java.util.Date(context.getStartupDate()));
```

## 6. Configuration with `getEnvironment()`

**`src/main/resources/application.properties`**

```properties
spring.application.name=employee-service
server.port=8081
```

```java
import org.springframework.core.env.Environment;

Environment env = context.getEnvironment();
System.out.println(env.getProperty("spring.application.name")); // employee-service
System.out.println(env.getProperty("server.port"));             // 8081
for (String profile : env.getActiveProfiles()) {
    System.out.println("Active profile: " + profile);
}
```

`Environment` combines multiple property sources; do not assume every property comes only from `application.properties`.

## 7. Loading files with `getResource()`

Create **`src/main/resources/employee.txt`**:

```text
Employee ID: 101
Employee Name: Ravi
```

Read it using the context:

```java
import java.nio.charset.StandardCharsets;
import org.springframework.core.io.Resource;

Resource resource = context.getResource("classpath:employee.txt");
String content = resource.getContentAsString(StandardCharsets.UTF_8);
System.out.println(content);
```

### `getResources()` — load multiple matching files

```java
import org.springframework.core.io.Resource;

Resource[] resources = context.getResources("classpath*:*.properties");
for (Resource resource : resources) {
    System.out.println(resource.getDescription());
}
```

The `classpath*:` prefix searches across classpath locations; actual matches depend on project dependencies and resource layout.

## 8. Internationalization with `getMessage()`

Create **`src/main/resources/messages.properties`**:

```properties
employee.welcome=Welcome to Employee Management System
```

```java
import java.util.Locale;

String message = context.getMessage("employee.welcome", null, Locale.ENGLISH);
System.out.println(message);
```

Expected output:

```text
Welcome to Employee Management System
```

A missing message code without a default can raise `NoSuchMessageException`.

## 9. Application events with `publishEvent()`

**`EmployeeCreatedEvent.java`**

```java
package com.example.demo.event;

public record EmployeeCreatedEvent(Long employeeId) {}
```

**`EmployeeEventListener.java`**

```java
package com.example.demo.event;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class EmployeeEventListener {
    @EventListener
    public void handle(EmployeeCreatedEvent event) {
        System.out.println("Employee created: " + event.employeeId());
    }
}
```

Publish after the context has started:

```java
context.publishEvent(new EmployeeCreatedEvent(101L));
```

Expected output:

```text
Employee created: 101
```

Listeners are synchronous by default with Spring's standard event multicaster, but configuration can change that behavior.

## 10. Programmatic wiring with `getAutowireCapableBeanFactory()`

```java
import org.springframework.beans.factory.config.AutowireCapableBeanFactory;

AutowireCapableBeanFactory factory = context.getAutowireCapableBeanFactory();
EmployeeService instance = factory.createBean(EmployeeService.class);
instance.display();
```

`createBean()` constructs and initializes a new instance using Spring wiring; it does **not** simply retrieve the existing singleton. Prefer ordinary dependency injection in application code.

## 11. BeanFactory vs ApplicationContext

| Capability | BeanFactory | ApplicationContext |
|---|---|---|
| Core bean retrieval and DI | Yes | Yes |
| Scope support | Yes | Yes |
| Message resolution | Not on base interface | Yes |
| Event publishing | Not on base interface | Yes |
| Resource loading | Not on base interface | Yes |
| Environment access | Not on base interface | Yes |
| Bean enumeration | Through `ListableBeanFactory` | Yes |
| Common entry point in Spring Boot | Underlying container | Yes |

## 12. Best practices

1. **Prefer constructor injection** for ordinary services and controllers.
2. Use `getBean()` for genuinely dynamic lookup, infrastructure integrations, or diagnostics.
3. Do not assume a single bean exists for every type; multiple implementations can make type-based lookup ambiguous.
4. Avoid depending on internal bean-definition counts or names for business logic.
5. Understand singleton versus prototype lifecycle before using scope inspection methods.
6. Treat the application context as an infrastructure API, not as a global service locator.

**Preferred constructor-injection example:**

```java
import org.springframework.stereotype.Component;

@Component
public class EmployeeRunner {
    private final EmployeeService employeeService;

    public EmployeeRunner(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    public void run() {
        employeeService.display();
    }
}
```

## 13. Interview questions

**Q1. What is `ApplicationContext`?**  
It is Spring's application-level IoC container interface for accessing managed beans and services such as events, messages, resources, and environment configuration.

**Q2. What is the difference between `getBean()` and constructor injection?**  
`getBean()` looks up a dependency programmatically; constructor injection declares the dependency explicitly and is generally preferred.

**Q3. Why might `getBean(EmployeeService.class)` fail?**  
No matching bean may exist, or multiple matching beans may make the request ambiguous.

**Q4. Does `getBeanDefinitionCount()` count only my `@Service` beans?**  
No. It counts registered bean definitions, including framework definitions, in the current context.

**Q5. What is the difference between `containsBean()` and `containsBeanDefinition()`?**  
`containsBean()` checks availability under a name; `containsBeanDefinition()` checks for a locally registered definition.

**Q6. How do you read a property?**  
Call `context.getEnvironment().getProperty("key")`.

**Q7. How do you publish an application event?**  
Call `context.publishEvent(event)` and handle it using an `@EventListener` method.

**Q8. Is `ApplicationContext` the same as `BeanFactory`?**  
No. `ApplicationContext` extends core bean factory capabilities with application-oriented features.

## 14. Student practice assignments

1. Print all bean definition names and count them.
2. Create two implementations of a `PaymentService` interface; inspect names using `getBeanNamesForType()`.
3. Create a prototype bean and compare `isSingleton()` and `isPrototype()`.
4. Read a custom `app.course.name` property using `getEnvironment()`.
5. Load a text file from `src/main/resources` using `getResource()`.
6. Publish an `OrderPlacedEvent` and handle it with `@EventListener`.
7. Rewrite a `getBean()`-based example using constructor injection.

---

**Suggested environment:** Java 21, Spring Boot 3.x, Maven, no Lombok.  
**Official reference:** https://docs.spring.io/spring-framework/reference/core/beans/introduction.html
