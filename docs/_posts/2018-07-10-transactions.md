---
layout: post
title: Spring - Transaction Management
---
# Transaction Management

**Why use Transactions?** _To Enforce the **ACID** Principles_

- **A**tomic - Each unit of work is an all-or-nothing operation
- **C**onsistent - Database integrity constraints are never violated
- **I**solated - Isolating transactions from each other
- **D**urable - Committed changes are permanent

- **Local Transactions** – Single Resource - Transactions managed by underlying resource
- **Global** (distributed) **Transactions** – Multiple Resources - Transaction managed by separate, dedicated transaction manager

Spring uses the same API for global vs. local.

`PlatformTransactionManager` abstraction hides implementation details.

Spring’s `PlatformTransactionManager` is the base interface for the abstraction, possible implementations:

- `DataSourceTransactionManager`
- `JmsTransactionManager`
- `JpaTransactionManager`
- `JtaTransactionManager`
- `WebLogicJtaTransactionManager`
- `WebSphereUowTransactionManager`
- and more...

Turning on by annotatiooin on configuration class `@EnableTransactionManagement`

```java
@Configuration
@EnableTransactionManagement
public class TxnConfig {
    @Bean
    // bean name is 'transactionManager' important for Spring internals
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

Adding transactional behaviour by `@Transactional` annotation.

```java
public class RewardNetworkImpl implements RewardNetwork { @Transactional
public RewardConfirmation rewardAccountFor(Dining d) {
// atomic unit-of-work
} }
```

- Target object wrapped in a proxy. Uses an Around advice
- Proxy implements the following behavior
  - Transaction started before entering the method
  - Commit at the end of the method
  - Rollback by default if method throws a RuntimeException (can be overridden)
- Transaction context bound to current thread. 
- All controlled by configuration
