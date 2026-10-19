# Spring Bean Lifecycle - Notes and Java Examples

The bean lifecycle describes how Spring creates, prepares, exposes, and eventually destroys a managed object. Spring Boot uses Spring Framework's bean lifecycle.

**Example baseline:** Java 17 or later, Spring Framework 6 or later, and Java configuration. Use `jakarta.annotation` imports for these versions. Examples are independent unless explicitly connected.

## 1. Lifecycle flow

```text
Register bean definition
        |
Create instance (resolve constructor arguments before invoking constructor)
        |
Populate configured properties / inject fields and setters
        |
Aware callbacks and before-initialization processing
        |     @PostConstruct runs through a processor in this phase
        |
InitializingBean.afterPropertiesSet()
        |
Configured init method
        |
After-initialization processing (may expose an AOP proxy)
        |
Bean available for application use
        |
Managed destruction
        |
@PreDestroy -> DisposableBean.destroy() -> configured destroy method
```

This is the ordinary creation path, not every internal extension point. A bean definition is a recipe; registering it is different from creating its instance.

## 2. Callback choices

| Mechanism | Initialization | Destruction |
| --- | --- | --- |
| Jakarta annotations | `@PostConstruct` | `@PreDestroy` |
| Spring interfaces | `InitializingBean.afterPropertiesSet()` | `DisposableBean.destroy()` |
| Bean configuration | `@Bean(initMethod = "prepare")` | `@Bean(destroyMethod = "release")` |

For distinct callback methods, Spring invokes them in the order shown above: annotation, interface, configured method. Prefer one mechanism per responsibility; the next example combines them to demonstrate order. See [Spring lifecycle callbacks](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html).

## 3. Complete lifecycle demonstration

Create `BeanLifecycleDemo.java` in a project with `spring-context` and `jakarta.annotation-api` on the classpath. In an existing Spring Boot project, use its dependency management for versions. This standalone example does not require a web server.

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class BeanLifecycleDemo {

    public static class MessageCatalog {
        public String greeting() {
            return "Welcome to Spring";
        }
    }

    public static class GreetingService
            implements BeanNameAware, InitializingBean, DisposableBean {

        private MessageCatalog catalog;

        public GreetingService() {
            System.out.println("1. Constructor");
        }

        public void setCatalog(MessageCatalog catalog) {
            this.catalog = catalog;
            System.out.println("2. Dependency supplied through setter");
        }

        @Override
        public void setBeanName(String name) {
            System.out.println("3. BeanNameAware: " + name);
        }

        @PostConstruct
        public void validate() {
            if (catalog == null) {
                throw new IllegalStateException("MessageCatalog is required");
            }
            System.out.println("4. @PostConstruct");
        }

        @Override
        public void afterPropertiesSet() {
            System.out.println("5. InitializingBean.afterPropertiesSet");
        }

        public void prepare() {
            System.out.println("6. Configured init method");
        }

        public void greet() {
            System.out.println("7. Business method: " + catalog.greeting());
        }

        @PreDestroy
        public void beforeShutdown() {
            System.out.println("8. @PreDestroy");
        }

        @Override
        public void destroy() {
            System.out.println("9. DisposableBean.destroy");
        }

        public void release() {
            System.out.println("10. Configured destroy method");
        }
    }

    @Configuration
    public static class Config {
        @Bean
        public MessageCatalog messageCatalog() {
            return new MessageCatalog();
        }

        @Bean(initMethod = "prepare", destroyMethod = "release")
        public GreetingService greetingService(MessageCatalog catalog) {
            GreetingService service = new GreetingService();
            service.setCatalog(catalog);
            return service;
        }
    }

    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext(Config.class)) {
            context.getBean(GreetingService.class).greet();
        }
    }
}
```

Expected application output, excluding framework logs:

```text
1. Constructor
2. Dependency supplied through setter
3. BeanNameAware: greetingService
4. @PostConstruct
5. InitializingBean.afterPropertiesSet
6. Configured init method
7. Business method: Welcome to Spring
8. @PreDestroy
9. DisposableBean.destroy
10. Configured destroy method
```

The factory method explicitly calls the setter here. Spring resolves the factory method's `MessageCatalog` argument, then manages the returned service. Try-with-resources closes the context after `greet()`.

## 4. Practical annotation-based example

For an application component, constructor injection plus annotation callbacks keeps the code straightforward. Place this class in a component-scanned package. This small lookup table is a teaching example; it performs no external I/O.

```java
import java.util.HashMap;
import java.util.Map;
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class CountryDirectory {
    private final Map<String, String> names = new HashMap<>();

    @PostConstruct
    public void load() {
        names.put("IN", "India");
        names.put("JP", "Japan");
        System.out.println("Country directory initialized");
    }

    public String find(String code) {
        return names.getOrDefault(code, "Unknown");
    }

    @PreDestroy
    public void clear() {
        names.clear();
        System.out.println("Country directory cleared");
    }
}
```

Example interaction:

```text
Context starts:       Country directory initialized
find("IN"):          India
find("XX"):          Unknown
Context closes:      Country directory cleared
```

Clearing an in-memory map is illustrative, not necessary resource cleanup. Real cleanup often releases owned files, executors, or connections. Stop callers before clearing shared state.

## 5. Configured methods for a class without Spring annotations

Use this approach when adapting a class you cannot annotate. Save this independent example as `ClientConfig.java`.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ClientConfig {

    public static class LocalClient {
        private boolean open;

        public void connect() {
            open = true;
            System.out.println("Client opened");
        }

        public String read() {
            if (!open) {
                throw new IllegalStateException("Client is closed");
            }
            return "example data";
        }

        public void disconnect() {
            open = false;
            System.out.println("Client closed");
        }
    }

    @Bean(initMethod = "connect", destroyMethod = "disconnect")
    public LocalClient localClient() {
        return new LocalClient();
    }
}
```

With `ClientConfig` registered, `read()` returns `example data` after initialization. The `open` flag models a resource; this example opens no network connection.

## 6. BeanPostProcessor: observe initialization

A `BeanPostProcessor` operates on bean instances and can return wrappers. A `BeanFactoryPostProcessor` operates on configuration metadata before ordinary beans are instantiated. See [Spring container extension points](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html).

To extend section 3, add these imports:

```java
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.core.Ordered;
import org.springframework.core.PriorityOrdered;
```

Add this nested class inside `BeanLifecycleDemo`:

```java
public static class TraceProcessor
        implements BeanPostProcessor, PriorityOrdered {

    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String name) {
        if ("greetingService".equals(name)) {
            System.out.println("TRACE: before initialization");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String name) {
        if ("greetingService".equals(name)) {
            System.out.println("TRACE: after initialization");
        }
        return bean;
    }
}
```

Add this method inside `Config`:

```java
@Bean
public static TraceProcessor traceProcessor() {
    return new TraceProcessor();
}
```

For this configuration, the first trace appears between steps 3 and 4; the second appears between steps 6 and 7. The explicit priority places this processor before the standard annotation initialization processor. Without ordering, do not assume every before-initialization processor runs before `@PostConstruct`.

The static factory method allows early registration without constructing the enclosing configuration instance. Keep infrastructure processors independent of ordinary application services.

## 7. Scope changes lifecycle timing

| Scope | Typical creation timing | Automatic cleanup boundary |
| --- | --- | --- |
| Singleton | Context startup; lazy beans when needed | Context close |
| Prototype | Each container request for an instance | Caller manages cleanup |
| Request | First access within the request | Request completion |
| Session | First access within the session | Session expiration or invalidation |

Spring initializes prototypes but does not automatically invoke their destruction callbacks. See [Spring bean scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html).

**Experiment:** add `@Scope("prototype")` to `greetingService()` in section 3, with the import `org.springframework.context.annotation.Scope`. Steps 1-6 run when `getBean()` is called. Step 7 still runs, but steps 8-10 are absent when the context closes.

**Lazy singleton experiment:** instead add `@Lazy` with the import `org.springframework.context.annotation.Lazy`. Initialization moves to the first lookup in this demo; destruction still occurs on close. An eager dependent bean can cause a lazy dependency to be created earlier.

## 8. Bean initialization versus application startup

`@PostConstruct` prepares one bean; it does not mean the entire application is ready. `SmartLifecycle` coordinates start/stop behavior for managed components. Initialization callbacks run before the bean's ordinary AOP proxy is exposed, so do not depend on `@Transactional` interception of the initialization method itself. See [startup, shutdown, and initialization callbacks](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html).

For the directory example, loading a small local map fits initialization. A long-running consumer needs explicit start/stop coordination rather than an endless loop in `load()`.

## 9. Common mistakes

| Mistake | Correction |
| --- | --- |
| Reading an injected field inside the constructor | Prefer constructor parameters, or inspect injected state in initialization. |
| Calling `new CountryDirectory()` and expecting callbacks | Retrieve a managed bean; plain construction does not invoke Spring callbacks. |
| Assuming the constructor means initialization is complete | Compare steps 1 and 4 in the demonstration. |
| Registering the same class through both scanning and `@Bean` unintentionally | Choose one registration path for the intended instance. |
| Cleaning up the same resource in all three destruction mechanisms | Assign cleanup to one callback. |
| Assuming shutdown callbacks run after a forced process kill | Design recoverable resource handling; shutdown callbacks cannot cover crashes. |
| Using `javax.annotation` imports in these examples | Use `jakarta.annotation` and ensure its API dependency is available. |
| Closing resources owned by another component | Let the owning component manage their lifetime. |

If initialization allocates a resource and then fails, release that resource in the failure path; do not rely on normal destruction of a bean that never finished initialization.

## 10. Practice exercises

1. Remove `service.setCatalog(catalog)` from section 3. Identify the failing callback and explain why `greet()` is never reached.
2. Retrieve `GreetingService` twice. Predict how many constructor messages appear for singleton and prototype scope.
3. Add the trace processor and verify where its messages appear.
4. Convert `CountryDirectory` to configured init/destroy methods without annotation callbacks.
5. Explain why an object may be constructed successfully but still fail to become a usable bean.

Related notes: [Bean scopes](Spring_Boot_Bean_Scopes_Detailed_Notes.md) and [Spring container and dependency injection](Spring_Container_BeanFactory_ApplicationContext_Scopes_Lifecycle_Detailed_Notes.md).

**Validation note:** examples are written for the stated dependencies; they have not been compiled or executed in this notes workspace.
