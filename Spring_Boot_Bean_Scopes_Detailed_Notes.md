# Spring Bean Scopes — Notes and Java Examples

A bean scope determines when Spring creates an object and how long it reuses it. Spring Boot uses Spring Framework's scope rules.

## 1. Scope comparison

| Scope | Instance boundary | Example use |
| --- | --- | --- |
| `singleton` (default) | One per bean definition per container | Pricing service |
| `prototype` | New instance per container lookup or dependency resolution | Report builder |
| `request` | One per HTTP request | Request tracking |
| `session` | One per HTTP session | Shopping cart |
| `application` | One per `ServletContext` | Shared web application counter |
| `websocket` | One per STOMP WebSocket session | Chat connection state |

The last four require web infrastructure; WebSocket scope requires STOMP messaging setup. See the [official Spring scope reference](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html).

## 2. Runnable singleton and prototype example

Add this file to a Java project with `spring-context` available. In a Spring Boot project, let Boot manage Spring dependency versions.

**File: `ScopeDemo.java`**

```java
import java.util.UUID;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

public class ScopeDemo {

    static class PricingService {
        public int total(int quantity, int unitPrice) {
            return quantity * unitPrice;
        }
    }

    static class ReportBuilder {
        private final String id = UUID.randomUUID().toString();
        private final StringBuilder content = new StringBuilder();

        public void append(String text) {
            content.append(text);
        }

        public String build() {
            return id + ": " + content;
        }
    }

    static class ReportService {
        private final ObjectProvider<ReportBuilder> builders;

        ReportService(ObjectProvider<ReportBuilder> builders) {
            this.builders = builders;
        }

        public String create(String text) {
            ReportBuilder builder = builders.getObject();
            builder.append(text);
            return builder.build();
        }
    }

    @Configuration
    static class Config {
        @Bean
        PricingService pricingService() {
            return new PricingService();
        }

        @Bean
        @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
        ReportBuilder reportBuilder() {
            return new ReportBuilder();
        }

        @Bean
        ReportService reportService(ObjectProvider<ReportBuilder> builders) {
            return new ReportService(builders);
        }
    }

    public static void main(String[] args) {
        try (var context = new AnnotationConfigApplicationContext(Config.class)) {
            var pricing1 = context.getBean(PricingService.class);
            var pricing2 = context.getBean(PricingService.class);
            System.out.println("Same singleton: " + (pricing1 == pricing2));

            var builder1 = context.getBean(ReportBuilder.class);
            var builder2 = context.getBean(ReportBuilder.class);
            System.out.println("Same prototype: " + (builder1 == builder2));

            var reports = context.getBean(ReportService.class);
            System.out.println(reports.create("Sales report"));
            System.out.println(reports.create("Inventory report"));
        }
    }
}
```

Expected output (IDs vary):

```text
Same singleton: true
Same prototype: false
<first UUID>: Sales report
<second UUID>: Inventory report
```

### Why use ObjectProvider?

Directly injecting a prototype into a singleton resolves it once. `ObjectProvider.getObject()` requests a fresh prototype on demand.

```java
// This singleton retains the same builder across calls.
class ReusingReportService {
    private final ScopeDemo.ReportBuilder builder;

    ReusingReportService(ScopeDemo.ReportBuilder builder) {
        this.builder = builder;
    }

    public String create(String text) {
        builder.append(text);
        return builder.build();
    }
}
```

For example, calling `create("Sales")` followed by `create("Inventory")` on this object produces accumulated content: `SalesInventory`.

## 3. Request scope example

Place each public class below in its own file, in a package scanned by your servlet-based Spring Boot application. These examples require Spring Web MVC. They are component examples, not a complete web application.

```java
import java.util.UUID;
import org.springframework.stereotype.Component;
import org.springframework.web.context.annotation.RequestScope;

@Component
@RequestScope
public class RequestTrace {
    private final String id = UUID.randomUUID().toString();

    public String getId() {
        return id;
    }
}
```

```java
import org.springframework.stereotype.Service;

@Service
public class TraceService {
    private final RequestTrace trace;

    public TraceService(RequestTrace trace) {
        this.trace = trace;
    }

    public String describe() {
        return trace.getId() + " / " + trace.getId();
    }
}
```

Call `describe()` during an MVC request:

```text
Request A: aaa / aaa
Request B: bbb / bbb
```

`@RequestScope`, `@SessionScope`, and `@ApplicationScope` supply scoped proxies by default. A proxy routes calls to the current target. Accessing a request target without an active request fails.

## 4. Session scope example

```java
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import org.springframework.stereotype.Component;
import org.springframework.web.context.annotation.SessionScope;

@Component
@SessionScope
public class ShoppingCart {
    private final List<String> products = new CopyOnWriteArrayList<>();

    public void add(String product) {
        products.add(product);
    }

    public List<String> getProducts() {
        return List.copyOf(products);
    }
}
```

Example interaction:

```text
Session A, request 1: add("Book")  -> [Book]
Session A, request 2: add("Pen")   -> [Book, Pen]
Session B, request 1: getProducts() -> []
```

Use the same session cookie when trying successive requests. Two browser tabs commonly share a session. This cart demonstrates in-memory storage; it has no checkout or persistence behavior.

## 5. Application scope example

```java
import java.util.concurrent.atomic.AtomicLong;
import org.springframework.stereotype.Component;
import org.springframework.web.context.annotation.ApplicationScope;

@Component
@ApplicationScope
public class ApplicationCounter {
    private final AtomicLong count = new AtomicLong();

    public long increment() {
        return count.incrementAndGet();
    }
}
```

Example calls from different sessions in one web application:

```text
Session A: increment() -> 1
Session B: increment() -> 2
Session A: increment() -> 3
```

Singleton boundaries follow Spring containers; application scope follows the servlet context. Neither provides a cluster-wide instance.

## 6. WebSocket scope example

Use this component in an application already configured for Spring STOMP over WebSocket. Call it while processing a message for an active session.

```java
import java.util.UUID;
import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.ScopedProxyMode;
import org.springframework.stereotype.Component;

@Component
@Scope(value = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ChatConnection {
    private final String id = UUID.randomUUID().toString();

    public String getId() {
        return id;
    }
}
```

```text
Connection A, message 1: getId() -> aaa
Connection A, message 2: getId() -> aaa
Connection B, message 1: getId() -> bbb
```

## 7. Lifecycle and common mistakes

Prototype initialization runs, but automatic destruction callbacks do not; callers own cleanup. Other scopes support destruction at their lifecycle boundaries. Scope does not guarantee thread safety. See [Spring's lifecycle and scope explanation](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html).

| Mistake | Practical correction |
| --- | --- |
| Keeping the current customer's name in a singleton service field | Pass customer information as method arguments. |
| Expecting a new prototype on every method call | Obtain one builder per operation through `ObjectProvider`. |
| Comparing proxy identities to identify request targets | Compare values returned by the target, such as `getId()`. |
| Using `final` classes with class-based scoped proxies | Keep proxied classes and intercepted methods non-final. |
| Using request state directly in a background job | Copy the needed values into the job's input. |
| Treating session state as permanent storage | Persist data that must survive session expiration. |
| Assuming a scope annotation registers a bean | Also use component scanning or an `@Bean` definition. |

## 8. Practice exercises

1. Remove `@Scope` from `reportBuilder()` and predict the identity comparison.
2. Call `ReportService.create()` three times and compare the generated IDs.
3. Add a second `PricingService` bean definition and retrieve both beans by name.
4. Try the cart from two independent browser sessions and compare the products.
5. Explain why the counter example uses `AtomicLong` rather than `long`.

Related notes: [Spring container, scopes, and lifecycle](Spring_Container_BeanFactory_ApplicationContext_Scopes_Lifecycle_Detailed_Notes.md).
