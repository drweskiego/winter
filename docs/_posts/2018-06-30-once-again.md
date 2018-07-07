---
layout: post
title: Spring - DI
---
# Spring - DI

## Java based configuration

``` java
@Configuration
public class ApplicationConfig {

    @Bean
    @Description("Handles all book related use-cases")
    public BookService bookService() {
        return new BookServiceImpl(bookRepository());
    }

    @Bean
    @Description("Provides access to data from the Books table")
    // @Autowired DataSource also possible
    public BookRepository bookRepository(DataSource dataSource) {
        return new JdbcBookService(dataSource);
    }
}

@Configuration
@Import(ApplicationConfig.class )
public class TestInfrastructureConfig {
    @Bean
    @Description("Data-source for the underlying RDB we are using")
    public DataSource dataSource() {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName("org.postgresql.Driver");
        ds.setUrl("jdbc:postgresql://localhost/database_name");
        ds.setUsername("maras");
        ds.setPasswort("s3cret");
    }
}
```

`@Configuration` is a **bean**

```java
// create spring application from config
ApplicationContext context = SpringApplication.run( ApplicationConfig.class );

// No need for bean id if type is unique
TransferService ts1 = context.getBean(TransferService.class );
// Use typed method to avoid cast
TransferService ts2 = context.getBean(“transferService”, TransferService.class);
// Classic way: cast is needed
// name from method name id no defined in @Bean
TransferService ts3 = (TransferService) context.getBean(“transferService”);
```

It is **not illegal** to define the same bean more than once

```java
 @Import({ Config1.class, Config2.class }) public class TestApp {
public class TestApp { 
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(TestApp.class);

        // expected Id=example1
        System.out.println("Id=" + context.getBean("example"));
    }
}

@Configuration
@Order(2)
public class Config1 {
    @Bean
    public String example() {
        return new String("example1");
    }
}

@Configuration
@Order(1)
public class Config2 {
    @Bean
    public String example() {
        return new String("example2");
    }
}
```


## Environment

```java
@Autowired
public TransferServiceImpl(@Value("${daily.limit}") int max) {
    this.maxTransfersPerDay = max; 
}

@Autowired
public void setDailyLimit(@Value("${daily.limit}") int max) {
    this.maxTransfersPerDay = max; 
}

```

## SpEL

```java
@Value("#{environment['daily.limit']}") int maxTransfersPerDay;
```

## Annotation configuration

```java
@Configuration
@ComponentScan ( { "com.bank.app.repository", "com.bank.app.service", "com.bank.app.controller" } )
public class AnnotationConfig {
    // No bean definition needed any more
}

@Component("transferService")
@Scope("prototype")
// @Scope(scopeName = "prototype")
@Profile("dev")
@Lazy(false)
public class TransferServiceImpl implements TransferService {

    @Autowired
    private Optional<AccountService> accountService;
    // accountService.ifPresent( s -> {...});

    @Autowired(required=false)
    public void setAccountRepository(
        @Qualifier("jdbcAccountRepository") AccountRepository a
        ) { this.accountRepository = a; }

    @Autowired
    public TransferServiceImpl (@Qualifier("jpaAccountRepository") AccountRepository accRep) {}
    // equal to 
    public TransferServiceImpl (AccountRepository jpaAccountRepository) {}

    // any visibility, no args, void, JSR-250
    // In javax.annotation package
    // after dependency injection
    @PostConstruct
    void populateCache() { }

    // any visibility, no args, void, JSR-250
    // In javax.annotation package
    // prior to destroj instance
    @PreDestroy
    void flushCache() { }
}
```

equals to

```java
@Configuration
public class TransferConfiguration {
    @Bean(name="transferService")
    @Scope("prototype")
    // @Scope(scopeName = "prototype")
    @Profile("dev")
    @Lazy(false)
    public TransferService tsvc() {
        return new TransferServiceImpl( accountRepository());
    }
}

public class TransferServiceImpl implements TransferService {

    public TransferServiceImpl (AccountRepository jpaAccountRepository) {}

    // any visibility, no args, void, JSR-250
    // In javax.annotation package
    // after dependency injection
    @PostConstruct
    void populateCache() { }

    // any visibility, no args, void, JSR-250
    // In javax.annotation package
    // prior to destroj instance
    @PreDestroy
    void flushCache() { }
}
```

### constr vs setter

Constructors            |Setters
---                     | ---
Mandatory dependencies  | Optional/changeable dependencies
Immutable dependencies  | Circular dependencies
Concise (pass several params at once)| Inherited automatically
                        |If constructor needs too many params

### java vs annotation

#### java

- Pros
      – Write any Java code you need
  - Strong type checking enforced by compiler (and IDE) 
  – Can be used for all classes (not just your own)
- Cons
  - More verbose than annotations
  – Is centralized in one (or a few) places

#### annotation

- Pros
  - Nice for your own beans
  – Single place to edit (just the class) 
  – Allows for very rapid development
- Cons
  – Configuration spread across your code base 
  - Harder to debug/maintain
  - Only works for your own code
  - Merges configuration and code (bad sep. of concerns)

## Scopes

--- | ---
singleton|A single instance is used
prototype|A new instance is created each time the bean is referenced
session|A new instance is created once per user session - web environment only
requestA new instance is created once per request – web environment only

## Lifecycle methods

```java
@Component("transferService")
public class TransferServiceImpl implements TransferService {

    // any visibility, no args, void, JSR-250
    // after dependency injection (constructor & setter)
    @PostConstruct
    void populateCache() { }

    // any visibility, no args, void, JSR-250
    // prior to destroj instance, no prototype, no killed process
    @PreDestroy
    void flushCache() { }
}
```

equals to

```java
@Configuration
public class TransferConfiguration {
    @Bean(name="transferService", initMethod="populateCache", destroyMethod="flushCache")
    public TransferService tsvc() {
        return new TransferServiceImpl( accountRepository());
    }
}

public class TransferServiceImpl implements TransferService {

    public TransferServiceImpl (AccountRepository jpaAccountRepository) {}

    // @PostConstruct also works
    void populateCache() { }

    // @PreDestroy also works
    void flushCache() { }
}
```

Annotations commes from **JSR-250** <br>
Package `javax.annotation`

Order
- constructor
- setters
- @PostConstruct

@PretDestroy
- not called for **prototype**
- during `ConfigurableApplicationContext` closing
- uses JVM shutdown hook

```java
 ConfigurableApplicationContext context = SpringApplication.run(...);
 //...

 // calls all @PreDestroy
 context.close();
 ```

## stereotypes

 - `@Component` stareotypes:
   - `@Service` - service klases
   - `@Repository` - database access
   - `@Controller` - web MVC
   - `@RestController` - web MVC
   - `@Configuration` - java configuration

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Service
@Transactional(timeout=60)
public @interface MyTransactionalService {
    String value() default "";
}
```

`@Service` is used to annotate other annotation - is caled meta-annotation
