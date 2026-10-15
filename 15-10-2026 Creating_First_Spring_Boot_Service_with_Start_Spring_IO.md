# Creating Your First Spring Boot Service with start.spring.io

## 1. What you will build

In this exercise, you will create a small Spring Boot web service that exposes a REST endpoint:

```text
GET http://localhost:8080/hello
```

The endpoint will return a simple response such as:

```text
Hello, Spring Boot!
```

You will create the project using [start.spring.io](https://start.spring.io/), open it in an IDE, understand the generated files, run the application, and test the service.

## 2. Required software

Install the following before starting:

| Software | Recommendation |
|---|---|
| Java Development Kit | Java 21 LTS is a good choice for a new project. Java 17 is the minimum supported version for current Spring Boot releases. |
| IDE | IntelliJ IDEA, Spring Tool Suite, Eclipse, or Visual Studio Code with Java extensions |
| Build tool | Maven or Gradle. The generated project includes a Maven/Gradle wrapper, so a separate installation is optional for normal project commands. |
| Browser or API client | A browser, curl, Postman, or similar tool for testing the endpoint |

Check Java from a terminal:

```bash
java -version
javac -version
```

The commands should report the JDK you intend to use. If `javac` is not found, a JRE may be installed instead of a complete JDK.

## 3. Java and Spring Boot version selection

### Java version

For a new application, choose **Java 21** when it is available in the Initializr list. It is an LTS release and is a convenient default for learning and production projects.

Choose **Java 17** when:

- Your organization standardizes on Java 17.
- Your course, deployment environment, or existing libraries require Java 17.
- You want the broadest compatibility with Spring Boot 3.x projects.

Do not select a Java version higher than the version installed on your machine. For example, selecting Java 21 while running only a Java 17 JDK can cause a build or runtime error.

### Spring Boot version

On the date of these notes, start.spring.io lists the Spring Boot 4.1.1 line as the latest stable release. Select the latest **stable, non-snapshot** version displayed by Initializr when you create a new project.

Avoid versions marked with:

- `SNAPSHOT`: an in-development build that may change.
- `M1`, `M2`, or another milestone label: a pre-release build.

Spring Boot 4.x requires Java 17 or newer. If you are following an older course or joining an existing Boot 3 project, select the exact Boot version used by that project instead of changing it casually. Spring Boot 3.x also requires Java 17 or newer.

Always check the generated `pom.xml` or `build.gradle` after downloading so you know exactly which version was selected.

## 4. Open start.spring.io

1. Open [https://start.spring.io/](https://start.spring.io/).
2. Use the form to define the project.
3. Add the required dependency.
4. Click **Generate** to download a ZIP file.

## 5. Configure the project in Initializr

Use these values for the first service:

| Initializr field | Value | Meaning |
|---|---|---|
| Project | Maven | Uses Maven and creates a `pom.xml` file. Gradle is also valid, but this note uses Maven commands. |
| Language | Java | The source code will be Java. |
| Spring Boot | Latest stable non-snapshot version | Use the stable version shown by Initializr. |
| Group | `com.example` | The organization or ownership namespace. |
| Artifact | `employee-command-service` | The project/application name. |
| Name | `employee-command-service` | Usually filled automatically from Artifact. |
| Description | `First Spring Boot REST service` | A short description of the project. |
| Package name | `com.example.employeecommandservice` | The base Java package for the application. |
| Packaging | `Jar` | Creates an executable JAR with an embedded web server. |
| Java | `21` or `17` | Must match a JDK installed on your machine. |

### Understanding Group ID, Artifact ID, and package name

These three values are related but have different purposes:

- **Group ID**: identifies the organization or logical owner. For example, `com.example` or `com.mycompany`.
- **Artifact ID**: identifies this application or library. For example, `employee-command-service`.
- **Package name**: identifies the Java namespace used in source files. For example, `com.example.employeecommandservice`.

A common relationship is:

```text
Group ID       = com.example
Artifact ID    = employee-command-service
Package name   = com.example.employeecommandservice
```

Package names normally use lowercase letters and dots. Do not use spaces, hyphens, or Java keywords in a package name. The artifact may contain hyphens, but the Java package should use a valid identifier such as `employeecommandservice`.

## 6. Add the dependency

Click **Add Dependencies** and select:

### Spring Web

This dependency is required for a basic HTTP/REST service. It provides Spring MVC, an embedded web server, JSON support, and the annotations used by controllers.

For this first exercise, no database dependency is required. You can add validation, Spring Data JPA, security, or other dependencies later when the application needs them.

## 7. Generate and open the project

1. Click **Generate**.
2. Save the downloaded ZIP file.
3. Extract it into a development folder, for example:

   ```text
   C:\projects\employee-command-service
   ```

4. Open the extracted project folder in your IDE.
5. Allow the IDE to import the Maven project and download dependencies.

The generated structure will look similar to this:

```text
employee-command-service/
├── .mvn/
├── src/
│   ├── main/
│   │   ├── java/com/example/employeecommandservice/
│   │   │   └── EmployeeCommandServiceApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/example/employeecommandservice/
│           └── EmployeeCommandServiceApplicationTests.java
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md
```

## 8. Understand the main application class

Initializr creates a main class similar to this:

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

### What this class does

- `main` is the normal Java entry point.
- `SpringApplication.run(...)` starts Spring and the embedded web server.
- `@SpringBootApplication` combines configuration, component scanning, and auto-configuration.
- Component scanning starts from the package containing this class and searches its subpackages.

Keep the main class in the root package, such as `com.example.employeecommandservice`. Put controllers and services in subpackages so Spring can discover them automatically.

## 9. Create the first REST controller

Create this file:

```text
src/main/java/com/example/employeecommandservice/EmployeeCommandController.java
```

Add the following code:

```java
package com.example.employeecommandservice;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmployeeCommandController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

### Controller explanation

- `@RestController` tells Spring that this class handles HTTP requests and that returned values should be written directly to the response body.
- `@GetMapping("/hello")` maps an HTTP GET request to `/hello`.
- The `hello()` method returns the response text.

## 10. Run the application

### Run from the IDE

Open `EmployeeCommandServiceApplication.java`, then click the Run button beside the `main` method or run the class as a Java application.

### Run with the Maven wrapper on Windows

From the project root, run:

```powershell
.\mvnw.cmd spring-boot:run
```

### Run with the Maven wrapper on macOS/Linux

```bash
./mvnw spring-boot:run
```

The wrapper is preferred because it uses the Maven version configured for the project.

### Build and run the JAR

Create a packaged application:

```powershell
.\mvnw.cmd clean package
```

Run the generated JAR:

```powershell
java -jar target/employee-command-service-0.0.1-SNAPSHOT.jar
```

The exact JAR filename can be checked inside the `target` directory.

## 11. Confirm that the application started

The console should contain a message indicating that the application has started and that the server is listening on port `8080`.

By default, Spring Boot uses:

```text
http://localhost:8080
```

Test the endpoint in a browser by opening:

```text
http://localhost:8080/hello
```

Or use curl:

```bash
curl http://localhost:8080/hello
```

Expected response:

```text
Hello, Spring Boot!
```

## 12. Change the server port

If port `8080` is already in use, add this line to:

```text
src/main/resources/application.properties
```

```properties
server.port=8081
```

Restart the application and use:

```text
http://localhost:8081/hello
```

## 13. Run the generated test

The generated project includes a context-load test. Run it with:

```powershell
.\mvnw.cmd test
```

This verifies that the Spring application context can start successfully. A passing build normally ends with `BUILD SUCCESS`.

## 14. Common problems and solutions

### `java` or `javac` is not recognized

Install a JDK and configure `JAVA_HOME` and the system `PATH`. Restart the terminal after changing environment variables.

### Unsupported class file version

The project was compiled for a newer Java version than the JDK being used. Check both the Initializr Java setting and `java -version`, then use a matching JDK.

### Port 8080 is already in use

Stop the other application or set `server.port` to another port such as `8081`.

### The endpoint returns 404

Check that:

1. The URL is exactly `/hello`.
2. The controller has `@RestController`.
3. The method has `@GetMapping("/hello")`.
4. The controller is in the same package as the main class or in one of its subpackages.
5. The application was restarted after the file was created.

### Maven cannot download dependencies

Check the internet connection, proxy settings, and Maven repository access. Reopen the project and allow the IDE to finish importing dependencies.

## 15. Final checklist

- [ ] A supported JDK is installed and `java -version` works.
- [ ] A stable Spring Boot version was selected in Initializr.
- [ ] The Group ID is meaningful, such as `com.example`.
- [ ] The Artifact ID identifies the service, such as `employee-command-service`.
- [ ] The package name is valid and uses lowercase names.
- [ ] Packaging is `Jar`.
- [ ] `Spring Web` is selected.
- [ ] The project was generated, extracted, and opened in an IDE.
- [ ] The main application class contains `@SpringBootApplication`.
- [ ] A controller maps `GET /hello`.
- [ ] The application starts without errors.
- [ ] `http://localhost:8080/hello` returns the expected response.
- [ ] The Maven test passes.

## 16. Official references

- [Spring Initializr](https://start.spring.io/)
- [Spring Boot system requirements](https://docs.spring.io/spring-boot/system-requirements.html)
- [Spring Boot installation guide](https://docs.spring.io/spring-boot/installing.html)
