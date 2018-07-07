---
layout: post
title: Once again
---
# Spring

## Java based configuration

``` java
@Configuration
public class ApplicationConfig {
    @Bean
    public BookService bookService() {
        return new BookServiceImpl(bookRepository());
    }

    @Bean
    public BookRepository bookRepository() {
        return new JdbcBookService(dataSource());
    }

    @Bean
    public DataSource dataSource() {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName("org.postgresql.Driver");
        ds.setUrl("jdbc:postgresql://localhost/database_name");
        ds.setUsername("maras");
        ds.setPasswort("s3cret");
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
@Profile("dev")
@Lazy(false)
public class TransferServiceImpl implements TransferService {

    @Autowired
    private Optional<AccountService> accountService;
    // accountService.ifPresent( s -> {...});

    @Autowired(required-false)
    public void setAccountRepository(@Qualifier("jdbcAccountRepository") AccountRepository a) { this.accountRepository = a;}

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