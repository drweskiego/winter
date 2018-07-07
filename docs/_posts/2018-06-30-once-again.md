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

## SpEL

## Annotation configuration

```java
@Configuration
@ComponentScan ( “com.bank” )
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
    public TransferServiceImpl (@Qualifier("jpaAccountRepository") AccountRepository accRep) { ... }
    // equal to 
    public TransferServiceImpl (AccountRepository jpaAccountRepository)
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
```

### constr vs setter

Constructors            |Setters
---                     | ---
Mandatory dependencies  | Optional/changeable dependencies
Immutable dependencies  | Circular dependencies
Concise (pass several params at once)| Inherited automatically
                        |If constructor needs too many params

