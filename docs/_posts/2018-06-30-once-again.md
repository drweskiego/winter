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
    // DataSource autowired from other configuration automatically
    // alternatively @Autowired can be used on field etc.
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

`@Configuration` annotated class is a **bean**

```java
// create spring application from config
ApplicationContext context = SpringApplication.run( ApplicationConfig.class );

// No need for bean id if type is unique
TransferService ts1 = context.getBean(TransferService.class );
// Use typed method to avoid cast
TransferService ts2 = context.getBean("transferService", TransferService.class);
// Classic way: cast is needed
// name from method name id no defined in @Bean
TransferService ts3 = (TransferService) context.getBean("transferService");
```

It is **not illegal** to define the same bean more than once

```java
@Configuration
@Import({ Config1.class, Config2.class })
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

Properties from many sources:

- System Environment Variables - auto
- JVM System Properties - auto
- Java Properties Files
- Servlet Context Parameters
- JNDI

Avaliable in:

- `@Bean` methods
- component properties (any visibility)
- `@Autowired`  setters methods
- `@Autowored` constructors

Default file: `app.properties`

```ini
db.driver=org.postgresql.Driver
db.url=jdbc:postgresql:localhost/transfer
db.user=transfer-app
db.password=secret45
 ```

```java
@Configuration
@PropertySource("classpath:/com/organization/config/app.properties")
@PropertySource("file:config/local-${ENV}.properties")
public class DbConfig {
    Environment env;

    @Autowired
    public DbConfig(Environment env) {
        this.env = env; 
    }

    @Bean
    public DataSource dataSource(
            @Value("${db.url}") String url
            @Value("#{environment['db.user']}") String user) { 
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName( env.getProperty("db.driver"));
        ds.setUrl(url);
        ds.setUser(user);
        ds.setPassword( env.getProperty( "db.password" ));
        return ds;
    }
}

@Autowired
public TransferServiceImpl(@Value("${daily.limit}") int max) {
    this.maxTransfersPerDay = max;
}

@Autowired
// fallback 10000
public void setDailyLimit(@Value("${daily.limit : 10000}") int max) {
    this.maxTransfersPerDay = max;
}
```

## SpEL

default references: `environment`, `systemProperties`, `systemEnvironment`

```java
@Configuration
class TaxConfig {
    // fallback 10000
    @Value("#{environment['daily.limit'] ?: 10000}") int maxTransfersPerDay;

    @Bean
    public TaxCalculator taxCalculator2 (@Value("#{ systemProperties['user.region'] }") String region) {
        return new TaxCalculator( region ); 
    }
}

@Configuration
class AnotherConfig {
    @Value("#{taxCalculator2.methodName}") KeyGenerator kgen;
    @Value("#{new java.net.URL(environment['home.page']).host}") String hostname;
}
```

[http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)

## Profiles

```java
@Configuration
@Profile("jpa")
@PropertySource ("dev.properties")
public class DataSourceConfig {
    @Bean(name="dataSource")
    @Profile("dev")
    public DataSource dataSourceForDev() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        return builder.setName("testdb");
    }
    @Bean(name="dataSource")
    @Profile("!dev")
    public DataSource dataSourceForProd() {
        BasicDataSource dataSource = new BasicDataSource(); ...
        return dataSource;
    }
}
```

Define profile:

- command line: `-Dspring.profiles.active=dev,jpa`
- code: `System.setProperty("spring.profiles.active", "dev,jpa"); SpringApplication.run(AppConfig.class);`
- test: `@ActiveProfiles`

## Annotation configuration

```java
@Configuration
@ComponentScan ( { "com.bank.app.repository", "com.bank.app.service", "com.bank.app.controller" } )
public class AnnotationConfig {
    // No bean definition needed any more
}

@Component("transferService")
// @Scope(scopeName = "prototype")
@Scope("prototype")
@Profile("dev")
@Lazy(false)
public class TransferServiceImpl implements TransferService {

    @Autowired
    private Optional<AccountService> accountService;
    // accountService.ifPresent( s -> {...});

    @Autowired(required=false)
    public void setAccountRepository(@Qualifier("jdbcAccountRepository") AccountRepository a) {
        this.accountRepository = a;
    }

    @Autowired
    public TransferServiceImpl (@Qualifier("jpaAccountRepository") AccountRepository accRep) {}
    // equal to
    @Autowired
    public TransferServiceImpl (AccountRepository jpaAccountRepository) {}
    // equal to
    @Resource("jpaAccountRepository")
    public TransferServiceImpl (AccountRepository accRep) {}

}
```

equals to

```java
@Configuration
public class TransferConfiguration {
    @Bean(name="transferService")
    // @Scope(scopeName = "prototype")
    @Scope("prototype")
    @Profile("dev")
    @Lazy(false)
    public TransferService tsvc() {
        return new TransferServiceImpl( accountRepository());
    }
}

public class TransferServiceImpl implements TransferService {

    public TransferServiceImpl (AccountRepository jpaAccountRepository) {}

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

scope | desc
--- | ---
**singleton**  | Lasts as long as its ApplicationContext
**prototype**  | A new instance is created each time the bean is referenced, lasts as long as you refer to it, then garbage collected
**session**    | A new instance is created once per user session - web environment only, lasts as long as user's HTTP session
**request**    | A new instance is created once per request – web environment only, lasts as long as user's HTTP request
**application**| Lasts as long as the ServletContext (Spring 4.0)
**global**     | Lasts as long as a global HttpSession in a Portlet application (_obsolete from Spring 5_)
**thread**     | Lasts as long as its thread – defined in Spring but not registered by default
**websocket**  | Lasts as long as its websocket (Spring 4.2)
**refresh**    | Can outlive reload of its application context. Difficult to do well, assumes Spring Cloud Configuration Server

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

## stereotype annotations

 - `@Component` root stereotype
 - `@Service` - service
 - `@Repository` - database access
 - `@Controller` - web MVC
 - `@Indexed` - ...
 - `@RestController` - web MVC compossed adnotation - out of stereotypes package
 - `@Configuration` - java configuration - out of stereotypes package

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Service
@Transactional(timeout=60)
public @interface MyTransactionalService {
    String value() default "";
}
```

`@Service` is used to annotate other annotation - is caled **meta-annotation**

## DI implementation

Java Configuration uses CGLIB for inheritance-based subclasses

CGLIB does not support classes with constructors, Spring now uses Objenesis internally to overcome this

```java
@Configuration
public class AppConfig {
    @Bean public AccountRepository accountRepository() { ... }
    @Bean public TransferService transferService() { ... }
}

// autogenerated CGLIB
public class AppConfig$$EnhancerByCGLIB$ extends AppConfig { 
    public AccountRepository accountRepository() {
        // if bean is in the applicationContext, then return bean
        // else call super.accountRepository(), store bean in context, return bean
    }
    public TransferService transferService() {
        // if bean is in the applicationContext, then return bean
        // else call super.transferService(), store bean in context, return bean
    }
}
```

## Factories

```java
public class AccountServiceFactoryBean implements FactoryBean <AccountService>
{
    // never call directly, Spring does this
    public AccountService getObject() throws Exception {
    // Conditional logic – for example: selecting the right
    // implementation or sub-class of AccountService to create return accountService;
    }
    public Class<?> getObjectType() { return AccountService.class; }
    // isSingleton defaults to returning true since Spring V5 }
}

@Configuration
public class ServiceConfig {
    @Bean
    public AccountServiceFactoryBean accountService() {
       return new AccountServiceFactoryBean();
    }

    @Bean
    //  getObject() called by Spring internally
    public CustomerService customerService(AccountService accountService) {
        return new CustomerService(accountService);
    }
}
```