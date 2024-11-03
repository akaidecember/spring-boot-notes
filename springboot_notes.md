# Spring Boot Notes for Interview

### Things to cover / study

    • Go through all annotations
    • qualifier annotation
    • cosmos db as a dependency
    • maven
    • actuator 
    • dependency injection 
    • docker kubernetes 
    • spring security hibernate
    • MVC (Model Vieww Controller)
    • threading framework
    • spring services - netflix oss etl stack - a lot of stuff, famous 
    • hystrics
    • circuit breaker pattern 
    • spring cloud config server
    • security - outh2 bearer token, awt token, someone can ask a lot so be careful 
    • distributed protocol - saga pattern, two face protocol
    • Solid principles 
    • oops
    • ELK stack 
    • spring cloud stream
    • event driven - kafka - queue based messaging systems 
    • zipkin and sleuth
    • zoul - server side load balancing 
    • resilience 4j used by more than hysteics 
    • spring batch 
    • splunk / zipkin
    • junit
    • JVM tuning and performance optimization techniques

Read all of the above topics and throw these keywords to show competence.

---

# Spring Boot Annotations

Spring Boot made configuring Spring much easier with it's _auto-configuration_ feature.

* ## **@SpringBootApplication**

    We use this annotation to mark the main class of the Spring Boot Application. This is the entry point for a Spring Boot application, as Java requires main class to exist as a starting point.

    The annotation **`@SpringBootApplication`** includes **`@Configuration`**, **`@EnableAutoConfiguration`** and _@ComponentScan_ with their default attributes.

    ```java
    @SpringBootApplication
    class Application{
        public static void main(String[] args){
            SpringApplication.run(Application.class, args);
        }
    }
    ```

* ## **@EnableAutoConfiguration**

    As it name says, it enables auto-configuration. It means that Spring Boot looks for auto-coinfiguration beans on it's class path and automatically applies them.

    _Note that we have to use this annotation with **`@Configuration`** annotation_

    ```java
    @Configuration
    @EnableAutoCOnfiguration
    class ApplicationConfig{
        // ... code ...
    }
    ```

**Note - Usually, when we write our custom auto-configurations, we want Spring to use them conditionally. We can achieve this with the annotations below. We can place the annotations on _@Configuration_ classes or _@Bean_ methods.**

* ## **@ConditionalOnClass** and **@ConditionalOnMissingClass**

    Using these conditions, Spring will only use the marked auto-configuration bean if the class in the annotation's argument is present / absent.

    These annotations are contitional annotations that control the bean creation based on the presence / absence of specific classes on the class path.

  * The **`@ConditionalOnClass`** configuration is used to create a bean if only a specified class is present on the class path. Commonly used in auto-configuration classes to ensure that certain beans are only created if related dependencies are available.

        Example:

        ```java
        @Configuration
        @ConditionalOnClass(DataSource.class)
        class MySQLConfig{
            // ... code ...
        }
        ```

  * The **`@ConditionalOnMissingClass`** annotation is used to create a bean only if a specified class is not present on the classpath.

        Example:

        ```java
        @Configuration
        @ConditionalOnMissingClass("com.example.SomeLibrary")
        public class FallbackConfig {
            @Bean
            public FallbackService fallbackService() {
                // Configure and return FallbackService bean
            }
        }
        ```

* ## **@ConditionalOnBean** and **@ConditionalOnMissingBean**

    These annotations are used to conditionally create beans based on the presence or absence of other specific beans in the Spring context.

  * **`@ConditionalOnBean`**: This annotation creates a bean only if another specific bean is present in the context.
    * **Usage**: Useful when you want to set up dependent configurations that rely on a particular bean's presence.
    * **Example**:

        ```java
        @Configuration
        @ConditionalOnBean(name = "dataSource")
        public class DataConfig {
            // Configuration that depends on dataSource bean
        }
        ```

  * **`@ConditionalOnMissingBean`**: This annotation creates a bean only if a specific bean is _not_ present in the context.
    * **Usage**: Often used to provide default configurations that can be overridden by a user-defined bean.
    * **Example**:

        ```java
        @Configuration
        @ConditionalOnMissingBean(FallbackService.class)
        public class DefaultConfig {
            @Bean
            public FallbackService fallbackService() {
                return new FallbackService();
            }
        }
        ```

* ## **@ConditionalOnProperty**

    `@ConditionalOnProperty` controls bean creation based on the presence and value of a configuration property in the application properties file.

  * **Usage**: Useful for enabling or disabling beans based on externalized configurations.
  * **Example**: If you want to create a bean only if a feature is enabled via a property:

      ```java
      @Configuration
      @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
      public class FeatureConfig {
          // Configuration for feature
      }
      ```

    In this example, `FeatureConfig` will only be loaded if `feature.enabled=true` in the application properties.

* ## **@ConditionalOnResource**

    `@ConditionalOnResource` is used to create a bean only if a specific resource (like a file) is available on the classpath.

  * **Usage**: Useful for configurations that depend on specific files being present, such as property files or configuration files.
  * **Example**:

      ```java
      @Configuration
      @ConditionalOnResource(resources = "classpath:my-config.yaml")
      public class ConfigLoader {
          // Configuration that depends on my-config.yaml
      }
      ```

    In this example, `ConfigLoader` will only be loaded if `my-config.yaml` exists on the classpath.

* ## **@ConditionalOnWebApplication** and **@ConditionalOnNotWebApplication**

    These annotations create beans based on whether the application is a web application or not.

  * **`@ConditionalOnWebApplication`**: Creates a bean only if the application is a web application (i.e., running with a `WebApplicationContext`).
    * **Example**:

        ```java
        @Configuration
        @ConditionalOnWebApplication
        public class WebConfig {
            // Web-specific configuration
        }
        ```

  * **`@ConditionalOnNotWebApplication`**: Creates a bean only if the application is _not_ a web application.
    * **Example**:

        ```java
        @Configuration
        @ConditionalOnNotWebApplication
        public class NonWebConfig {
            // Non-web-specific configuration
        }
        ```

* ## **@ConditionalOnExpression**

    `@ConditionalOnExpression` uses a Spring Expression Language (SpEL) expression to conditionally create a bean.

  * **Usage**: Provides more complex conditions than other annotations, allowing for custom expressions.
  * **Example**: To conditionally load a bean if a property has a specific value:

      ```java
      @Configuration
      @ConditionalOnExpression("${feature.enabled} == true && ${feature.version} >= 2")
      public class AdvancedFeatureConfig {
          // Advanced feature configuration
      }
      ```

    Here, `AdvancedFeatureConfig` will only be loaded if both conditions in the expression are met.

* ## **@Conditional**

    `@Conditional` is a more generic conditional annotation that allows for complex, custom conditions. It requires implementing a custom `Condition`.

  * **Usage**: Allows for conditions that cannot be achieved with simpler annotations.
  * **Example**:

      ```java
      @Configuration
      @Conditional(MyCustomCondition.class)
      public class CustomConfig {
          // Custom configuration
      }
      ```

  * **Custom Condition**: To use `@Conditional`, you need to create a class implementing `Condition`:

      ```java
      public class MyCustomCondition implements Condition {
          @Override
          public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
              // Custom logic here
              return true; // or false based on condition
          }
      }
      ```

* ## **@Configuration**

    The `@Configuration` annotation in Spring marks a class as a source of bean definitions for the application context. It tells Spring that the class can be used to define and configure beans, which are managed components of a Spring application, facilitating dependency injection and service orchestration.

    * **Usage**: Use on classes that define beans or configuration settings.
    * **Example**:
      ```java
      @Configuration
      public class AppConfig {
          @Bean
          public MyService myService() {
              return new MyService();
          }
      }
      ```

* ## **@ComponentScan**

    The `@ComponentScan` annotation in Spring tells the framework where to look for components, services, and configurations. It automatically discovers and registers beans in the specified packages, eliminating the need for manual bean registration and making it easier to manage and scale the application's architecture.

    * **Usage**: Use in combination with `@Configuration` to define the packages to scan for Spring-managed components.
    * **Example**:
      ```java
      @Configuration
      @ComponentScan(basePackages = "com.example")
      public class AppConfig { }
      ```

* ## **@Bean**

    The `@Bean` annotation in Spring marks a method in a configuration class to define a bean. This bean is then managed by the Spring container, which handles its lifecycle and dependencies. The `@Bean` annotation is used to explicitly create and configure beans that Spring should manage.

    * **Usage**: Place on methods in configuration classes to create and expose beans.
    * **Example**:
      ```java
      @Bean
      public MyService myService() {
          return new MyService();
      }
      ```

* ## **@Component**

    The `@Component` annotation in Spring marks a class as a Spring-managed component. This allows Spring to automatically detect and register the class as a bean in the application context, enabling dependency injection and making the class available for use throughout the application.

    * **Usage**: Place on classes to make them available as Spring beans.
    * **Example**:
      ```java
      @Component
      public class MyComponent { }
      ```

* ## **@Repository**

    The `@Repository` annotation in Spring marks a class as a data access component, specifically for database operations. It provides additional benefits like exception translation, making it easier to manage database access and integrate with Spring's data access framework.

    * **Usage**: Use on classes that interact with the database, such as DAO classes.
    * **Example**:
      ```java
      @Repository
      public class MyRepository {
          // Database access methods
      }
      ```

* ## **@Service**

    The `@Service` annotation in Spring marks a class as a service layer component, indicating that it holds business logic. It is used to create Spring-managed beans, making it easier to organize and manage services within the application.

    * **Usage**: Place on classes that contain business logic or service-related functionality.
    * **Example**:
      ```java
      @Service
      public class MyService {
          // Business logic methods
      }
      ```

    **Cross-Question**: Can we use `@Component` instead of `@Repository` and `@Service`? If yes, then why do we use `@Repository` and `@Service`?
   
    Yes, we can use `@Component` instead of `@Repository` and `@Service` since all three create Spring beans. However, `@Repository` and `@Service` make our code clearer by showing the purpose of each class. `@Repository` also helps manage database errors better. Using these specific annotations makes our code easier to understand and maintain.

* ## **@Controller**

    The `@Controller` annotation in Spring marks a class as a web controller that handles HTTP requests. It is used to define methods that respond to web requests, show web pages, or return data, making it a key part of Spring's web application framework.

    * **Usage**: Place on classes that handle web requests and render views.
    * **Example**:
      ```java
      @Controller
      public class MyController {
          @RequestMapping("/home")
          public String home() {
              return "home";
          }
      }
      ```

* ## **@RestController**

    The `@RestController` annotation in Spring marks a class as a RESTful web service controller. It combines `@Controller` and `@ResponseBody`, meaning the methods in the class automatically return JSON or XML responses, making it easy to create REST APIs.

    * **Usage**: Place on classes that provide RESTful endpoints.
    * **Example**:
      ```java
      @RestController
      public class MyRestController {
          @GetMapping("/api/data")
          public Data getData() {
              return new Data();
          }
      }
      ```

* ## **@RequestMapping**

    The `@RequestMapping` annotation in Spring maps HTTP requests to handler methods in controller classes. It specifies the URL path and the HTTP method (GET, POST, etc.) that a method should handle, enabling routing and processing of web requests in a Spring application.

    * **Usage**: Use to define the URL and HTTP method for controller methods.
    * **Example**:
      ```java
      @RequestMapping(value = "/home", method = RequestMethod.GET)
      public String home() {
          return "home";
      }
      ```

* ## **@Autowired**

    The `@Autowired` annotation in Spring enables automatic dependency injection. It tells Spring to automatically find and inject the required bean into a class, reducing the need for manual wiring and simplifying the management of dependencies within the application.

    * **Usage**: Place on fields, setters, or constructors to inject dependencies.
    * **Example**:
      ```java
      @Autowired
      private MyService myService;
      ```

* ## **@PathVariable**

    The `@PathVariable` annotation in Spring extracts values from URI templates and maps them to method parameters. It allows handlers to capture dynamic parts of the URL, making it possible to process and respond to requests with path-specific data in web applications.

    * **Usage**: Use in controller methods to bind URL path parameters.
    * **Example**:
      ```java
      @GetMapping("/user/{id}")
      public User getUser(@PathVariable("id") Long id) {
          return userService.findById(id);
      }
      ```

* ## **@RequestParam**

    The `@RequestParam` annotation in Spring binds a method parameter to a web request parameter. It extracts query parameters, form data, or any parameters in the request URL, allowing the handler method to process and use these values in the application.

    * **Usage**: Use in controller methods to capture query parameters.
    * **Example**:
      ```java
      @GetMapping("/search")
      public List<User> search(@RequestParam("name") String name) {
          return userService.findByName(name);
      }
      ```

* ## **@ResponseBody**

    The `@ResponseBody` annotation in Spring tells a controller method to directly return the method's result as the response body, instead of rendering a view. This is commonly used for RESTful APIs to send data (like JSON or XML) back to the client.

    * **Usage**: Place on methods to return data as a response.
    * **Example**:
      ```java
      @ResponseBody
      @GetMapping("/data")
      public Data getData() {
          return new Data();
      }
      ```

* ## **@RequestBody**

    The `@RequestBody` annotation in Spring binds the body of an HTTP request to a method parameter. It converts the request body into a Java object, enabling the handling of data sent in formats like JSON or XML in RESTful web services.

    * **Usage**: Use in controller methods to capture and parse request bodies.
    * **Example**:
      ```java
      @PostMapping("/user")
      public User createUser(@RequestBody User user) {
          return userService.save(user);
      }
      ```

* ## **@EnableWebMvc**

    The `@EnableWebMvc` annotation in Spring activates the default configuration for Spring MVC. It sets up essential components like view resolvers, message converters, and handler mappings, providing a base configuration for building web applications.

* ## **@EnableAsync**

    The `@EnableAsync` annotation in Spring enables asynchronous method execution. It allows methods to run in the background on a separate thread, improving performance by freeing up the main thread for other tasks.

* ## **@Scheduled**

    The `@Scheduled` annotation in Spring triggers methods to run at fixed intervals or specific times. It enables scheduling tasks automatically based on cron expressions, fixed delays, or fixed rates, facilitating automated and timed execution of methods.

* ## **@EnableScheduling**

    `@EnableScheduling` is an annotation in Spring Framework used to enable scheduling capabilities for methods within a Spring application. It allows methods annotated with `@Scheduled` to be executed based on specified time intervals or cron expressions.

---
---

# QUESTIONS

* ## **What is Spring Boot?**

    Spring Boot is a powerfull framework that streamlines the development, testing and deployment of Spreing Applications. It eliminates the need for adding boilerplate code that is required for each java application to get up and running the project. It also offers automatic configuration features to ease the setup and integration of various development tools. 

* ## **What are the features of Spring Boot?**

    Key features of Spring Boot contain `auto-configuration`, which automatically sets up an application components based on the libraries present, embedded servers like Tomcat (default), Jetty etc. for deployment, a wide array of starter kits that bundle dependencies for specific functionalities, a complete monitoring system with Spring Boot actuator (**Used for monitoring Spring Boot application**), and extensive support for cloud environments, simplifying the development of cloud native applications.

* ## **Key components of Spring Boot**

    Key components:

    * `Spring Boot starter kits` - bundle dependencies for specific features
    * `Spring Boot AutoConfiguration` - automatically configures app based on the included dependencies
    * `Spring Boot CLI` - for dev. and testing apps from the CLI
    * `Spring Boot Actuator` - provides prod. ready features like health checks and metrics

* ## **What are the Spring Boot Starter Dependencies?**

    Spring Boot starter dependencies are pre-made packages that help us easily add specific features to the App. 

    Example: `springboot-starter-web` helps to build web apps, `springboot-starter-data-jpa` helps with databases, `springboot-starter-security` adds security features for authentication and authorization. 

* ## **How does a Spring Boot application get Started?**

    A Spring Application typically starts by initializing a Spring ApplicationContext, which manages the beans and dependencies. In Spring Boot, this is often triggered by calling the SpringApplication.run() in the main method, whose class is annotated by `@SpringBootApplication`, as mentioned in the annotations section above. It also starts the embedded server if necessary.

* ## **What does `@SpringBootApplication` do internally?**

    The `@SpringBootApplication` combines `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`. This triggers Spring's auto-configuration mechanism to automatically configure the application based on it's included dependencies, scans for Spring components, and sets up configuration classes.

* ## **What is a container in Spring Boot?**

    In Spring Boot, a container refers to the Spring Inversion of Control container, which is responsible for creating, managing and configuring the application's beans and their dependencies. It provides the core of Spring Framework's dependency injection capabilities, which allows Spring to manage objects, their lifecycles and their interactions in an automated manner.

    There are two types of Containers:

    * `BeanFactory` : The most basic Spring Container, providing basic DI support, is lightweight and primarily used for minimal memory usage, lacks advanced features.

    * `ApplicationContext` : More advanced container that extends `BeanFactory`, providing additional features such as event propagation, declarative mechanisms for creating a bean, and integration with Spring's web applications. `ApplicationContext`is the default container used in Spring Boot.

    In a Spring Boot application, the Spring container is initialized when the application starts up. It scans for beans (components, services, repositories, and configurations) in the specified packages and configures them based on the application’s configuration (usually using annotations).

* ## **What is a Spring Bean?**

    Spring Bean is an object managed by the Spring IoC (Inversion of Control) container. Beans are the backbones of a Spring application and are created, configured and managed by the Spring Container.

    Beans are typically instances of classes that represent the core business logic and dependencies in a Spring Application, and they are ised to wire together differnet parts of the application.

    Beans are Configured via Annotations or XML: Beans can be defined in various ways.

    * Using the `@Component`, `@Service`, `@Repository`, or `@Controller` annotations for automatic detection and registration.
    * Using `@Bean` within a `@Configuration` class to manually create and configure beans.
    By defining beans in an XML configuration file (less common in modern Spring applications).

* ## **What is auto-wired?**

    **Auto-wiring** in Spring is a feature that allows Spring's IoC (Inversion of Control) container to automatically inject beans into other beans, managing their dependencies without explicit configuration. This feature simplifies the process of wiring together components by letting Spring resolve and inject the required dependencies based on specific rules.

    * ### How Auto-Wiring Works

        When you use auto-wiring, Spring examines the dependencies of a bean and automatically injects the required beans by matching them in the Spring context (container). It uses reflection and type-matching to inject the correct beans without the need for manual configuration.

    * ### Types of Auto-Wiring in Spring

        Spring provides several types of auto-wiring to control how dependencies are injected:

        **@Autowired (byType)**: The `@Autowired` annotation is used to mark a field, constructor, or setter method for auto-wiring by type.
        - **By Type**: Spring injects a bean based on the type of the dependency. For example, if a class has a `DataSource` field marked with `@Autowired`, Spring looks for a bean of type `DataSource` in the context.
        - **Example**:
            ```java
            @Service
            public class MyService {
                @Autowired
                private MyRepository myRepository;
            }
            ```

        **@Qualifier (byName)**: Sometimes, there may be multiple beans of the same type in the Spring container. In such cases, you can use the `@Qualifier` annotation with `@Autowired` to specify the bean name explicitly.
        - **By Name**: This method allows you to choose a specific bean by name when multiple beans of the same type are available.
        - **Example**:
            ```java
            @Service
            public class MyService {
                @Autowired
                @Qualifier("mySpecificRepository")
                private MyRepository myRepository;
            }
            ```

        **Constructor-Based Auto-Wiring**: `@Autowired` can also be used on constructors, making constructor injection a preferred way to wire dependencies, as it makes dependencies explicit.
        - **Example**:
            ```java
            @Service
            public class MyService {
                private final MyRepository myRepository;

                @Autowired
                public MyService(MyRepository myRepository) {
                    this.myRepository = myRepository;
                }
            }
            ```

        **Setter-Based Auto-Wiring**: `@Autowired` can also be applied to a setter method, which Spring will call to inject the dependency.
        - **Example**:
            ```java
            @Service
            public class MyService {
                private MyRepository myRepository;

                @Autowired
                public void setMyRepository(MyRepository myRepository) {
                    this.myRepository = myRepository;
                }
            }

    * ### Advantages of Auto-Wiring

        - **Reduces Boilerplate Code**: Auto-wiring minimizes the need for explicit configuration in XML or Java-based configuration files.
        - **Enhances Flexibility**: Dependencies can be injected automatically, allowing for easier modularity and code management.
        - **Improves Testability**: By using auto-wiring, it’s easier to inject mock dependencies for testing purposes.

    * ### Common Issues with Auto-Wiring

        - **Ambiguity**: If there are multiple beans of the same type and no `@Qualifier` is specified, Spring may throw an error due to ambiguity.
        - **No Matching Bean**: If no bean of the required type is available in the context, Spring will throw a `NoSuchBeanDefinitionException`.
        - **Circular Dependencies**: Auto-wiring can sometimes lead to circular dependencies, where two or more beans depend on each other.

    * ### Optional Dependencies

        If a dependency is not mandatory, you can use `required = false` to prevent Spring from throwing an error if the bean is not found:

        ```java
        @Autowired(required = false)
        private MyOptionalService optionalService;
        ```

* ## **What are ApplicationRunner and CommandLineRunner?**

    In Spring Boot, **ApplicationRunner** and **CommandLineRunner** are two interfaces that provide a way to execute code after the Spring Boot application has started. These interfaces are commonly used for tasks such as initialization, cleanup, or performing actions that need to occur immediately after application startup.

    ## **CommandLineRunner**

    `CommandLineRunner` is an interface with a single `run()` method that accepts a `String... args` parameter, which provides access to command-line arguments passed to the application. 

    - **Usage**: Implement the `CommandLineRunner` interface on a bean to execute code after the Spring application context is initialized and before the application is ready to serve requests.
    - **Example**:
        ```java
        import org.springframework.boot.CommandLineRunner;
        import org.springframework.stereotype.Component;

        @Component
        public class MyCommandLineRunner implements CommandLineRunner {
            @Override
            public void run(String... args) {
                System.out.println("CommandLineRunner executing with arguments: " + Arrays.toString(args));
                // Perform initialization or startup tasks here
            }
        }
        ```

    ## **Application Runner**

    In Spring Boot, **ApplicationRunner** is an interface that provides a way to execute code after the application has started. It is commonly used for initialization tasks, startup checks, or any actions that need to happen immediately after the application context is ready.

   * **When it Runs**: The `ApplicationRunner` interface runs after the Spring Boot application context has been fully initialized but before the application is ready to handle requests.

    * **Parameter**: It uses an `ApplicationArguments` parameter, which provides a structured way to access command-line arguments.

    * Example:

        To use `ApplicationRunner`, implement it in a bean by overriding the `run(ApplicationArguments args)` method. `ApplicationArguments` provides useful methods such as:
        - `getOptionNames()` - to get all option names (arguments starting with `--`).
        - `getOptionValues(String name)` - to get values for a specific option name.
        - `getNonOptionArgs()` - to get non-option arguments.

        ### Example of ApplicationRunner

        Here’s an example of implementing `ApplicationRunner` in a Spring Boot application:

        ```java
        import org.springframework.boot.ApplicationArguments;
        import org.springframework.boot.ApplicationRunner;
        import org.springframework.stereotype.Component;

        @Component
        public class MyApplicationRunner implements ApplicationRunner {
            @Override
            public void run(ApplicationArguments args) {
                System.out.println("ApplicationRunner executing with options: " + args.getOptionNames());
                System.out.println("Non-option arguments: " + args.getNonOptionArgs());
                
                // Perform any startup tasks or initialization here
            }
        }

* ## **What is Spring Boot dependency management?**

    **Spring Boot Dependency Management** is a feature provided by Spring Boot to simplify managing dependencies in a Spring application. Spring Boot comes with a built-in dependency management system that automatically manages versions and compatible dependencies, reducing the need to manually specify versions and ensuring that all components are compatible with each other.
    
    **Spring Boot dependency management** makes it easier to handle the dependencies that our project depends on. Instead of manually keeping their track, Spring Boot hels us manage them automatically. It uses tools like Maven or Gradle to organize these dependencies, making sure they work well together.

    ### Key Features of Spring Boot Dependency Management

    1. **Pre-Defined Dependency Versions**: Spring Boot provides a curated list of dependency versions for commonly used libraries through its "Spring Boot Starter Parent." When you use this parent in your project, you don’t need to specify versions for many popular libraries (like Spring Framework, Jackson, Hibernate, etc.), as they are managed and kept up-to-date by Spring Boot.

    2. **Compatible Dependencies**: The dependencies managed by Spring Boot are chosen and tested to work well together. This means that when you use Spring Boot starters or other dependencies, they are compatible with each other, reducing the risk of version conflicts.

    3. **Simplified BOM (Bill of Materials)**: Spring Boot includes a BOM (Bill of Materials), a file that specifies compatible versions of libraries. By using the BOM, you can override versions if needed without worrying about breaking compatibility.

    4. **Spring Boot Starters**: Spring Boot provides "starter" dependencies that bundle popular libraries for common functionality (e.g., `spring-boot-starter-web`, `spring-boot-starter-data-jpa`). These starters help quickly set up various parts of the application without worrying about individual dependencies and their versions.

    ### How Dependency Management Works

    When you create a Spring Boot project, you typically use the `spring-boot-starter-parent` in the `pom.xml` file (for Maven) or `spring-boot-dependencies` for Gradle. This provides a set of managed dependencies:

    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.3</version>
    </parent>


* ## **What happens if a starter dependency includes conflicting versions of libraries with other dependencies in the project?**

    If a Spring Boot starter dependency includes conflicting versions of libraries with other dependencies in the project, **dependency conflicts** or **version clashes** may occur. This can lead to unexpected behavior, runtime errors, or even failure to start the application. Spring Boot manages dependency versions, but conflicts can still happen when manually specified dependencies or other third-party libraries bring in versions incompatible with those provided by Spring Boot.

    ### Example Scenario of Dependency Conflict

    Consider a scenario where you’re using `spring-boot-starter-web` (which includes Spring MVC and Jackson for JSON processing) and add a dependency on a different version of Jackson directly in your project. This direct dependency might override the version provided by Spring Boot and cause version incompatibility issues.

    ```xml
    <dependencies>
        <!-- Spring Boot Starter Web (includes Jackson version managed by Spring Boot) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Direct Jackson dependency with a conflicting version -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.10.0</version> <!-- Potential conflict with Spring Boot’s version -->
        </dependency>
    </dependencies>
    ```

    In this example:

    * `spring-boot-starter-web` might include Jackson version 2.12.3 as per Spring Boot's managed version.
    * By adding jackson-databind version 2.10.0 manually, it introduces a conflicting version that may lack features or fixes present in the managed version.

    ### Causes of This Scenario
    * Manual Version Overrides: Adding a specific version of a library (like Jackson) that conflicts with the version provided by a Spring Boot starter.

    * Transitive Dependencies: Other libraries you add may include transitive dependencies with versions that conflict with Spring Boot’s managed versions.

    * External Libraries with Different Versions: Adding an external or legacy library that relies on an older or newer version of a dependency used by Spring Boot.

    ### Resolving Dependency Conflicts

    1. Exclusions: Use exclusions to remove a specific version of a dependency that conflicts with your Spring Boot starter dependency.

        ```xml
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>conflicting-library</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-databind</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        ```

    2. Dependency Management Section: Specify a consistent version for the conflicting dependency in the `<dependencyManagement> `section of your `pom.xml` to enforce a specific version.

        ```xml
        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-databind</artifactId>
                    <version>2.12.3</version> <!-- Align with Spring Boot’s managed version -->
                </dependency>
            </dependencies>
        </dependencyManagement>
        ```

    3. Check Effective POM: Use tools like mvn dependency:tree to see the full dependency tree and identify conflicting versions.

    Dependency conflicts in Spring Boot can lead to issues if different versions of the same library are included in the project. Identifying and resolving these conflicts is crucial to ensure compatibility and proper functioning of the application. Using dependency management best practices, like aligning versions and avoiding manual overrides, helps minimize such conflicts.

* ## **How can you disable specific auto-configuration class?**

    In Spring Boot, you can disable specific auto-configuration classes if you don’t want them to be applied in your application. This is often useful when a certain auto-configuration class interferes with custom configurations or causes unwanted behavior.

    ### _**Methods to Disable a Specific Auto-Configuration Class**_

    ### 1. Using @SpringBootApplication Exclude Attribute

    You can specify the auto-configuration classes to exclude directly in the @SpringBootApplication annotation using the exclude or excludeName attribute.

    ```java
    @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
    public class MyApplication {
        public static void main(String[] args) {
            SpringApplication.run(MyApplication.class, args);
        }
    }
    ```

    In this example, DataSourceAutoConfiguration is excluded, meaning Spring Boot will not automatically configure a DataSource for the application.

    ### 2. Using @EnableAutoConfiguration with Exclude Attribute
    If you are not using @SpringBootApplication or prefer more granular control, you can use the @EnableAutoConfiguration annotation with the exclude attribute.

    ```java
    Copy code
    @EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
    @Configuration
    public class MyConfig {
        // Custom configuration code
    }
    ```

    ### 3. Using Application Properties

    You can also disable auto-configuration by specifying the fully qualified class name in the application.properties file.

    ```properties
    spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
    ```

    Multiple classes can be excluded by separating them with a comma:

    ```properties
    spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
    ```

    ### Example Use Case
    Suppose you want to disable the DataSourceAutoConfiguration class because you’re managing database configuration manually. Disabling this auto-configuration prevents Spring Boot from automatically setting up a DataSource bean, allowing you to define a custom DataSource configuration.

    ### But why tho?

    Disabling specific auto-configuration classes can be helpful when:

    A particular auto-configuration conflicts with custom configurations.
    You want to optimize application startup by preventing unnecessary configurations.
    You have a specific requirement that deviates from Spring Boot's default configuration.
    By disabling only the necessary auto-configuration classes, you maintain the flexibility of Spring Boot’s auto-configuration while controlling specific configurations.

* ## **Describe the flow of HTTPS requests through a Spring Boot application**

    In a Spring Boot application, the flow of an HTTPS request goes through several stages, from receiving the request to sending the response back to the client. Here’s an overview of each step in the process:

    ### 1. **Client Sends an HTTPS Request**

    - The client (e.g., a web browser or REST client) initiates an HTTPS request to the Spring Boot application. 
    - Since it's HTTPS, the request is encrypted, providing a secure connection using SSL/TLS to protect data during transmission.

    ### 2. **SSL/TLS Handshake**

    - The server (Spring Boot application) has an SSL certificate configured. During the SSL/TLS handshake, the client and server exchange encryption keys to establish a secure, encrypted connection.
    - Once the handshake completes, the client can securely communicate with the server over HTTPS.

    ### 3. **Embedded Web Server (Tomcat, Jetty, or Undertow) Receives the Request**

    - In a Spring Boot application, an embedded web server (typically Tomcat by default) listens for incoming HTTPS requests on a specified port (usually 8443 for HTTPS).
    - The web server decrypts the HTTPS request and passes it to the Spring Boot application.

    ### 4. **DispatcherServlet Processes the Request**

    - Spring Boot’s **DispatcherServlet** is the central component that handles incoming HTTP/HTTPS requests. It’s part of the **Spring MVC** framework and acts as a front controller.
    - The `DispatcherServlet` intercepts the request and delegates it to the appropriate components in the application based on the URL and HTTP method.

    ### 5. **Handler Mapping Determines the Controller**

    - The `DispatcherServlet` uses **Handler Mapping** to determine which controller method should handle the request.
    - Spring Boot’s auto-configuration automatically maps requests to the appropriate controllers based on annotations like `@RequestMapping`, `@GetMapping`, and `@PostMapping`.

    ### 6. **Controller Handles the Request**

    - The appropriate controller method is invoked to process the request. Controller classes are annotated with `@RestController` or `@Controller` to define endpoints.
    - The controller receives the request parameters and any payload data (e.g., JSON), which Spring Boot converts into Java objects using `@RequestParam`, `@PathVariable`, or `@RequestBody`.

    ### 7. **Service Layer Processes Business Logic**

    - If the request requires additional processing, the controller calls methods from the **Service Layer**. Services, annotated with `@Service`, contain the business logic for the application.
    - The service may interact with other components, such as the data layer or external APIs, to process the request.

    ### 8. **Data Access Layer Interacts with the Database**

    - If data is needed from a database, the service calls methods in the **Data Access Layer** (DAOs or Repositories).
    - Repositories, often annotated with `DIY/@Repository`, use **Spring Data JPA** or another ORM framework to interact with the database.
    - The data is fetched or updated in the database, and the result is returned back up to the service layer.

    ### 9. **Controller Returns a Response**

    - After the service layer processes the business logic, the controller prepares a response. If the endpoint is a REST API, it usually returns a JSON or XML response.
    - The controller returns the response object, which Spring Boot automatically serializes (e.g., to JSON) using **Jackson** or another serializer.

    ### 10. **View Resolver (Optional)**

    - If the controller returns a view name (e.g., a web page), the **View Resolver** component resolves the view name to a specific template (such as a Thymeleaf or JSP page).
    - For REST APIs, this step is typically skipped as they return JSON or XML directly.

    ### 11. **DispatcherServlet Sends the Response**

    - The `DispatcherServlet` takes the response data from the controller and sends it back through the embedded web server.
    - If the application is configured to use HTTPS, the response is encrypted before it is sent to the client.

    ### 12. **Client Receives the HTTPS Response**

    - The client receives the encrypted response and decrypts it using the SSL/TLS session established earlier.
    - The client can then render the response (e.g., JSON data or a web page) as needed.

    The HTTPS request flow in a Spring Boot application is as follows:

    1. Client sends an HTTPS request.
    2. SSL/TLS handshake establishes a secure connection.
    3. Embedded web server receives and decrypts the request.
    4. DispatcherServlet processes the request.
    5. Handler Mapping finds the appropriate controller.
    6. Controller processes the request.
    7. Service layer executes business logic.
    8. Data access layer interacts with the database if needed.
    9. Controller returns the response.
    10. (Optional) View Resolver renders the view.
    11. DispatcherServlet sends the encrypted response.
    12. Client receives and decrypts the HTTPS response.

    This flow ensures a secure, organized, and modular way to process HTTPS requests in a Spring Boot application.

* ## **Explain the RestController annotation in springboot**

    The `@RestController` annotation is used to create RESTful web controllers. This annotation is a convinience annotation that combines `@Controller` and `@ResponseBody` annotation, which means that the data returned by each method will be written directly into the response body as JSON or XML, rather through view resolution.

    Used for - creating RESTful web services, it simplifies the creation of REST APIs by combining the functionality of `@Controller` and `@ResponseBody`.

    ```java
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @RestController
    @RequestMapping("/api")
    public class MyRestController {

        @GetMapping("/hello")
        public String sayHello() {
            return "Hello, World!";
        }

        @GetMapping("/user/{id}")
        public User getUserById(@PathVariable Long id) {
            // Simulate fetching a user based on ID
            return new User(id, "John Doe", "john.doe@example.com");
        }
    }
    ```

* ## **What is the difference between the RestController annotation and the controller annotation?**

    The key difference is that `@Controller` is used to mark classes as Spring MVC Controller and typically returns a view. 

    `@RestController` compbines the `@Controller` and `@ResponseBody`, indicating that all methods assume `@ResponseBody` by default, returning data instead of a view.

* ## **What is the differnece between RequestMapping and GetMapping?**

    @RequestMapping and @GetMapping are both annotations in Spring used to map HTTP requests to handler methods in a controller. However, they have some differences in terms of specificity and usage.

    ### 1. @RequestMapping
    * **Purpose**: @RequestMapping is a versatile annotation that can map multiple HTTP methods (GET, POST, PUT, DELETE, etc.) to a handler method.

    * **Supported HTTP Methods**: It can handle any HTTP method by specifying it with the method attribute (e.g., RequestMethod.GET, RequestMethod.POST).

    * **Usage**: @RequestMapping is often used when a single method needs to handle multiple HTTP methods or when you need a more generic mapping.

        Example:

        ```java
        @RequestMapping(value = "/user", method = RequestMethod.GET)
        public String getUser() {
            return "Get user";
        }

        @RequestMapping(value = "/user", method = RequestMethod.POST)
        public String createUser() {
            return "Create user";
        }
        ```
        
        In this example, @RequestMapping is used with method = RequestMethod.GET to handle GET requests and with method = RequestMethod.POST to handle POST requests.

    ### 2. @GetMapping
    * **Purpose**: @GetMapping is a specialized version of @RequestMapping that is specifically designed for handling HTTP GET requests.

    * **Supported HTTP Method**: It is specifically used for GET requests and cannot handle other HTTP methods (like POST, PUT, DELETE).

    * **Usage**: @GetMapping is more concise and is preferred when the method should handle only GET requests.

        Example:

        ```java
        @GetMapping("/user")
        public String getUser() {
            return "Get user";
        }
        ```

        In this example, @GetMapping is used to handle a GET request to /user.

    | Aspect               | @RequestMapping                                   | @GetMapping                         |
    |----------------------|---------------------------------------------------|-------------------------------------|
    | **HTTP Methods**     | Supports multiple HTTP methods                    | Only supports GET                   |
    | **Common Usage**     | General request mapping                           | GET-specific mapping                |
    | **Example Syntax**   | `@RequestMapping(value = "/url", method = RequestMethod.GET)` | `@GetMapping("/url")`              |
    | **Preferred Usage**  | When handling multiple HTTP methods               | When handling only GET requests     |


    ### When to Use Which?

    Use `@RequestMapping` when you need to handle multiple HTTP methods in the same method or if you need a more generic mapping.

    Use `@GetMapping` for GET requests when you want a concise, straightforward way to map GET requests specifically.

    In modern Spring applications, using @GetMapping, @PostMapping, @PutMapping, and @DeleteMapping is generally preferred for clarity, as they make the HTTP method explicit and concise.

* ## **How can you programmatically determine which profiles are currently active in a Spring Boot application?**

    You can programmatically determine which profiles are currently active in a Spring Boot application by using the Environment object provided by Spring. The Environment object contains information about the application’s environment, including the active profiles.

    ### Here are some ways to access the active profiles:

    ### 1. Using Environment in a Spring Component

    You can inject the Environment object into any Spring-managed component (like a service or controller) and call the getActiveProfiles() method to get the active profiles.

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.core.env.Environment;
    import org.springframework.stereotype.Component;

    @Component
    public class ProfileChecker {

        @Autowired
        private Environment environment;

        public void checkActiveProfiles() {
            String[] activeProfiles = environment.getActiveProfiles();
            System.out.println("Active Profiles: ");
            for (String profile : activeProfiles) {
                System.out.println(profile);
            }
        }
    }
    ```

    In this example, getActiveProfiles() returns an array of String values representing the currently active profiles.

    ### 2. Using Environment in the Main Application Class

    You can also check the active profiles directly in the main application class.

    ```java
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.core.env.Environment;

    @SpringBootApplication
    public class MyApplication implements CommandLineRunner {

        private final Environment environment;

        public MyApplication(Environment environment) {
            this.environment = environment;
        }

        public static void main(String[] args) {
            SpringApplication.run(MyApplication.class, args);
        }

        @Override
        public void run(String... args) throws Exception {
            String[] activeProfiles = environment.getActiveProfiles();
            System.out.println("Active Profiles: ");
            for (String profile : activeProfiles) {
                System.out.println(profile);
            }
        }
    }
    ```

    ### 3. Using ApplicationContext

    If you have access to the ApplicationContext, you can get the Environment from it and retrieve the active profiles.

    ```java
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;

    public class ProfileChecker {

        public static void main(String[] args) {
            ApplicationContext context = new AnnotationConfigApplicationContext(MyApplication.class);
            String[] activeProfiles = context.getEnvironment().getActiveProfiles();
            System.out.println("Active Profiles: ");
            for (String profile : activeProfiles) {
                System.out.println(profile);
            }
        }
    }
    ```

* ## **What is a Spring Boot actuator?**

    Spring Boot Actuator is a powerful set of tools and endpoints that provides insights into a Spring Boot application's health, metrics, and internal state. Actuator helps monitor and manage applications in production by exposing information about the application's runtime behavior and environment, making it essential for application management and operational tasks.

    ### Key Features of Spring Boot Actuator

    1. **Health Monitoring:** Actuator provides an /actuator/health endpoint that reports the health status of the application. It can be customized to check the status of external systems, such as databases, message brokers, or APIs that the application depends on.

    2. **Metrics**: Actuator provides metrics on various aspects of the application, such as memory usage, CPU usage, HTTP requests, database connections, and more. Metrics are accessible through the /actuator/metrics endpoint and can be configured to monitor custom metrics as well.

    3. **Auditing**: Actuator supports auditing by tracking application events, such as user login attempts or changes to important application state.

    4. **Environment and Configuration Properties**: Actuator exposes endpoints like `/actuator/env` and `/actuator/configprops` to view the application’s configuration properties, environment variables, and other properties set in the `application.properties` or `application.yml` file.

    5. **Thread Dump and Heap Dump**: Actuator provides tools for analyzing the state of threads and memory. The /actuator/threaddump endpoint shows the state of each thread, which is helpful in diagnosing performance issues. The /actuator/heapdump endpoint generates a memory dump to analyze memory usage.

    6. **Custom Endpoints**: Actuator allows you to create custom endpoints, enabling you to expose application-specific information that may not be covered by default endpoints.

    7. **Integration with Monitoring Systems**: Actuator integrates well with monitoring and alerting tools, such as Prometheus, Grafana, and Spring Boot Admin, making it easy to monitor application metrics and receive alerts.

    ### Common Actuator Endpoints

    Here are some of the most commonly used Actuator endpoints:

    | Endpoint               | Description                                                                                     |
    |------------------------|-------------------------------------------------------------------------------------------------|
    | `/actuator/health`     | Shows the health status of the application and connected resources (databases, services, etc.)  |
    | `/actuator/metrics`    | Provides various metrics about the application’s performance and resource usage                 |
    | `/actuator/info`       | Displays general application information, customizable via configuration                        |
    | `/actuator/env`        | Shows the environment properties, including system properties, environment variables, etc.      |
    | `/actuator/configprops`| Exposes configuration properties and their values, useful for debugging configuration           |
    | `/actuator/threaddump` | Provides a thread dump to analyze thread states                                                 |
    | `/actuator/heapdump`   | Generates a heap dump for memory usage analysis                                                 |
    | `/actuator/loggers`    | Allows viewing and changing logging levels dynamically                                          |
    | `/actuator/httptrace`  | Displays trace information for recent HTTP requests                                             |
    | `/actuator/mappings`   | Shows a list of all `@RequestMapping` paths mapped in the application                           |

    
    ### Enabling Actuator

    In a Spring Boot application, Actuator is enabled by adding the spring-boot-starter-actuator dependency:

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```

    Once Actuator is added, you can access the endpoints by navigating to /actuator in your browser or using a tool like curl.

    ### Securing Actuator Endpoints
    Since Actuator endpoints can expose sensitive information, it’s crucial to secure them, especially in production environments. You can secure Actuator endpoints by configuring Spring Security or limiting endpoint access to specific IP addresses.

    #### Example of securing endpoints in application.properties:

    ```properties
    management.endpoints.web.exposure.include=health,info  # Limit to specific endpoints
    management.endpoint.health.show-details=never          # Control health endpoint detail level
    ```

    ### Customizing Actuator

    1. **Custom Health Indicators:** You can add custom health checks for specific components in your application.
    
    2. **Custom Metrics:** You can define custom metrics using Micrometer, the metrics collection library used by Spring Boot Actuator.
    
    3. **Custom Endpoints:** Create custom endpoints by implementing Endpoint classes, allowing you to expose custom information or actions.

    Spring Boot Actuator is an invaluable tool for monitoring, managing, and understanding the behavior of a Spring Boot application in production. It provides extensive support for health monitoring, metrics, environment analysis, and diagnostics, making it essential for maintaining the health and performance of Spring Boot applications in production.

* ## **How to get a list of all the beans in the application?**

    You can get a list of all the beans in a Spring Boot application by accessing the `ApplicationContext` and calling its `getBeanDefinitionNames()` method. 
    
    Below are ways to do it:

    ### 1. Using `ApplicationContext` in a Component

    You can inject the `ApplicationContext` into any Spring-managed component (like a service or controller) and retrieve the bean names.

    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.ApplicationContext;
    import org.springframework.stereotype.Component;

    @Component
    public class BeanLister {

        @Autowired
        private ApplicationContext applicationContext;

        public void listAllBeans() {
            String[] beanNames = applicationContext.getBeanDefinitionNames();
            System.out.println("All Beans in Application Context:");
            for (String beanName : beanNames) {
                System.out.println(beanName);
            }
        }
    }
    ```

    ### 2. Listing Beans in the Main Application Class

    You can also list the beans in the main application class by getting the ApplicationContext returned by SpringApplication.run().

    ```java
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.ApplicationContext;

    @SpringBootApplication
    public class MyApplication {

        public static void main(String[] args) {
            ApplicationContext context = SpringApplication.run(MyApplication.class, args);
            String[] beanNames = context.getBeanDefinitionNames();

            System.out.println("All Beans in Application Context:");
            for (String beanName : beanNames) {
                System.out.println(beanName);
            }
        }
    }
    ```

    ### 3. Using Actuator’s /actuator/beans Endpoint

    If Spring Boot Actuator is enabled, you can also access a list of all beans via the /actuator/beans endpoint. This endpoint provides a JSON representation of all beans, their dependencies, and initialization order.

    * Add the Actuator dependency in your pom.xml or build.gradle:

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```

    * Enable the endpoint in application.properties:

    ```properties
    management.endpoints.web.exposure.include=beans
    ```

    * Access the endpoint at:

    ```bash
    http://localhost:8080/actuator/beans
    ```

* ## **Can we check the environment properties in our Spring Boot application? Explain how.**

    Yes, we can access environment properties in Spring Boot via the Environment interface. Inject the Environment into a bean using the `@Autowired` annotation and use the `getProperty()` method to retrieve the properties.

    ```java
    @Autowired
    private Environment env;

    String dbUrl = env.getProperty("database.url");
    ```

* ## **Explain the dev-tools dependency in Spring Boot**

    In Spring Boot, the DevTools dependency provides a set of tools that enhance the development experience by making it easier to test changes, monitor application behavior, and optimize productivity during development. The spring-boot-devtools dependency includes several features specifically designed to streamline and speed up the development workflow.

    ### Key Features and Benefits of spring-boot-devtools

    * #### **Automatic Restart:**

        DevTools enables automatic application restart when it detects changes in the classpath, like modifications to .java files.

        This feature eliminates the need to manually stop and restart the application every time you make a change, significantly improving productivity.

        The restart mechanism is fast because it only reloads changed classes rather than the entire application context.

    * #### **LiveReload Support:**

        DevTools includes LiveReload integration, which automatically refreshes the browser when static resources, such as HTML, CSS, or JavaScript files, are updated.

        For LiveReload to work, you need to install the LiveReload browser extension. This feature is especially useful in web applications for real-time testing of front-end changes.

    * #### **Property Overrides:**

        DevTools provides development-only properties that automatically override production settings when running in a development environment.

        For example, you can disable caching for templates (Thymeleaf, FreeMarker, etc.) and static resources during development to ensure you see your changes immediately.

        Some default properties provided by DevTools include:

        ```properties
        spring.thymeleaf.cache=false
        spring.freemarker.cache=false
        spring.groovy.template.cache=false
        spring.mustache.cache=false
        ```

    * #### **Automatic H2 Console and Other Dev Resources:**

        The spring-boot-devtools dependency enables easy access to the H2 console (if you're using the H2 in-memory database) and other development resources.
        These tools are only enabled in development mode to ensure they are not accidentally exposed in production.

    * #### **Remote Debugging:**

        DevTools provides support for remote debugging in scenarios where you want to test changes on a remote server (such as a development or staging environment) instead of your local machine.
        Remote debugging allows developers to push changes and reload the application on the server without restarting it entirely.
        
    * #### **Environment-Specific Configurations:**

        DevTools recognizes when the application is running in a development environment vs. a production environment and can automatically apply environment-specific configurations.

        For instance, DevTools is automatically disabled when running in a packaged application (like a .jar or .war file), ensuring that these features do not impact production performance.

    ### Adding spring-boot-devtools Dependency

    To enable DevTools, add the following dependency to your pom.xml (for Maven) or build.gradle (for Gradle):

    For Maven,

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
    </dependency>
    ```

* ## **What is the purpose of spting-boot-starter-parent?**

    The spring-boot-starter-parent is a special starter in Spring Boot that acts as a parent project to manage dependencies, configurations, and plugins for Spring Boot applications. It simplifies the configuration of a Spring Boot project by providing a set of pre-configured, default settings and conventions that are suitable for most applications.

    ### Key Purposes of spring-boot-starter-parent
    1. #### Dependency Management:

        `spring-boot-starter-parent` provides a bill of materials (BOM) for dependency versions. This means you don’t need to specify versions for many dependencies, as they are inherited from the parent.
        It automatically aligns dependencies to compatible versions tested with Spring Boot, which helps prevent version conflicts and ensures that libraries work seamlessly together.
        
        ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- Version is not specified, managed by spring-boot-starter-parent -->
        </dependency>
        ```

    2. #### Default Plugin Configuration:

        The spring-boot-starter-parent project pre-configures a variety of plugins, such as the Maven Compiler Plugin (for Java version configuration), Maven Surefire Plugin (for running tests), and others.

        These configurations simplify build management and set reasonable defaults for building and packaging Spring Boot applications.

    3. #### Predefined Build Settings:

        The parent provides sensible defaults for settings like encoding, test configurations, and resource filtering.

        For example, it defaults the character encoding to UTF-8 and Java source/target compatibility to version 1.8, making it easier to maintain consistency across projects.

        ```xml
        <properties>
            <java.version>1.8</java.version>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        </properties>
        ```

    4. #### Standardized Directory Structure:

        It enforces a standardized directory structure for source code, resources, and tests, which is a convention followed by Maven. This structure includes folders like src/main/java, src/main/resources, and src/test/java, making it easier for developers to understand and navigate the project.

    5. #### Simplified Plugin Management:

        `spring-boot-starter-parent` configures useful plugins such as the Spring Boot Maven Plugin, which simplifies tasks like packaging the application into an executable JAR or WAR and running it with `mvn spring-boot:run`.

        These plugins are configured with default options but can be customized as needed.

    6. #### Inheritance and Customization:

        By inheriting from spring-boot-starter-parent, you can override specific dependency versions, properties, and configurations if you need custom settings.

        If you don’t want to inherit the spring-boot-starter-parent as a parent, you can still get the same dependency management features by importing the spring-boot-dependencies BOM.

* ## **How do starters simplify the Maven or Gradle configuration?**

    Starters in Maven or Gradle simplify configuration by bundling common dependencies into a single
    package. Instead of manually specifying each dependency for a particular feature (like web
    development or JPA), we can add a starter (e.g.,`spring-boot-starter-web`), which includes all
    necessary libraries. This reduces configuration complexity, ensures compatibility, and speeds up the
    setup process, allowing developers to focus more on coding and less on dependency management.

* ## **How do you create REST APIs?**

    To create REST APIs in Spring Boot, annotate the class with `@RestController` and define methods
    with `@GetMapping`, `@PostMapping`, `@PutMapping`, or `@DeleteMapping` to handle HTTP requests. Then
    use `@RequestBody` for input data and `@PathVariable` or `@RequestParam` for URL parameters.  
    
    Then implement service logic and return responses as Java objects, which Spring Boot automatically
    converts to JSON. This setup handles API endpoints for CRUD operations.

* ## **What is versioning in REST? What are the ways that we can use to implement versioning?**

    Versioning in REST is a technique used to manage changes and updates to an API without breaking existing clients. As applications evolve, their APIs often need to change (e.g., adding new fields, renaming fields, or changing functionality), and versioning helps introduce these changes in a controlled manner. Versioning allows developers to release updates to the API while still supporting older versions, providing backward compatibility for clients using previous versions.

    * Ways to Implement Versioning in REST APIs

    There are several ways to implement versioning in REST APIs:

    * ### URI Versioning (Path Parameter)

        In this approach, the API version is included in the URI path, making it explicit in each request URL.

        Example:
        ```bash
        GET /api/v1/products
        GET /api/v2/products
        ```

        * Pros:
            * Clear and easy to understand.
            * Easy for caching mechanisms to distinguish between versions.
        * Cons:
            * Requires updating all URIs for each new version, which can be less flexible.
            * Doesn’t follow the RESTful principle that each resource should have a single URI.
        
    * ### Query Parameter Versioning

        In this approach, the API version is specified as a query parameter in the URL.

        Example:
        ```bash
        GET /api/products?version=1
        GET /api/products?version=2
        ```

        * Pros:
            * Simple to implement.
            * Allows the version to be optional (if not provided, the default version can be used).
        * Cons:
            * May not be ideal for caching, as query parameters are sometimes ignored in caching mechanisms.

* ## **What are the uses of repsonseEntity?**

    **`ResponseEntity`** in Spring is a powerful class that represents the entire HTTP response, including the status code, headers, and body. It provides a flexible way to build responses for RESTful APIs, allowing you to customize the response structure, set HTTP status codes, manage headers, and include data in the body.

    ### Key Uses of `ResponseEntity`

    1. **Setting HTTP Status Codes**

    `ResponseEntity` allows you to specify any HTTP status code for the response, enabling more precise control over response semantics. This is particularly useful for REST APIs where different status codes are required to indicate various outcomes, such as success, errors, or resource not found.

    ```java
    @GetMapping("/resource/{id}")
    public ResponseEntity<Resource> getResource(@PathVariable Long id) {
        Optional<Resource> resource = resourceService.findById(id);
        if (resource.isPresent()) {
            return ResponseEntity.ok(resource.get()); // 200 OK
        } else {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null); // 404 Not Found
        }
    }
    ```

    2. **Setting HTTP Headers**

    You can set custom headers in the response using `ResponseEntity`, which can be useful for adding metadata (like cache control, location, or authentication tokens) or to handle CORS policies.

    ```java
    @PostMapping("/upload")
    public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
        String location = fileStorageService.storeFile(file);
        return ResponseEntity.ok()
                .header("Location", location)
                .body("File uploaded successfully");
    }
    ``` 

    3. **Returning Response Bodies**

    `ResponseEntity` allows you to include a body in the response. This can be any object, which Spring automatically serializes (e.g., to JSON or XML) based on the content type and `Accept` header of the client request.

    ```java
    @GetMapping("/user/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findUserById(id);
        return ResponseEntity.ok(user); // Return user data as JSON
    }
    ```

    4. **Handling Empty Responses**

    `ResponseEntity` is useful when you need to return an empty response body, for example, for operations like deletion where no content is necessary in the response body. You can use `ResponseEntity<Void>` or `ResponseEntity.noContent()` for such scenarios.

    ```java
    @DeleteMapping("/user/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build(); // 204 No Content
    }
    ```

    5. **Error Handling and Custom Error Messages**

    `ResponseEntity` can be used to handle and respond with custom error messages and codes. This is especially useful in building REST APIs where you need to give meaningful error responses to clients.

    ```java
    @GetMapping("/account/{id}")
    public ResponseEntity<?> getAccount(@PathVariable Long id) {
        try {
            Account account = accountService.findAccount(id);
            return ResponseEntity.ok(account);
        } catch (AccountNotFoundException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND)
                    .body("Account not found with id " + id);
        }
    }
    ```

    6. **Conditional Responses**

    You can use `ResponseEntity` to send different responses based on the logic in your application. For example, returning different responses based on certain conditions, such as an authenticated user or an invalid request.

    ```java
    @GetMapping("/data")
    public ResponseEntity<String> getData(@RequestParam String param) {
        if (param.isEmpty()) {
            return ResponseEntity.badRequest().body("Parameter is required");
        }
        return ResponseEntity.ok("Here is the data for " + param);
    }
    ```

    ### Common `ResponseEntity` Methods

    | Method                          | Description                                                        |
    |---------------------------------|--------------------------------------------------------------------|
    | `ResponseEntity.ok()`           | Returns a 200 OK status with or without a body.                    |
    | `ResponseEntity.status()`       | Sets a custom HTTP status (e.g., 404, 500) and returns a response. |
    | `ResponseEntity.noContent()`    | Returns a 204 No Content status.                                   |
    | `ResponseEntity.badRequest()`   | Returns a 400 Bad Request status.                                  |
    | `ResponseEntity.created()`      | Returns a 201 Created status, often with a `Location` header.      |
    | `ResponseEntity.notFound()`     | Returns a 404 Not Found status.                                    |


    In all, `ResponseEntity` is a versatile class in Spring Boot that enhances control over HTTP responses, including status codes, headers, and body content. It is especially useful in REST APIs, as it allows you to build flexible, structured responses to handle various scenarios like success, error handling, empty responses, and conditional responses.


