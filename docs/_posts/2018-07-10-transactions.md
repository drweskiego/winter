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

Spring’s `PlatformTransactionManager` is the base interface for the abstraction

- DataSourceTransactionManager
- JmsTransactionManager
- JpaTransactionManager
- JtaTransactionManager
- WebLogicJtaTransactionManager
- WebSphereUowTransactionManager
- and more...

```java
@Bean
// bean name is 'transactionManager' important for Spring internals
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```

Other possible managers: `JtaTransactionManager`, `WebLogicJtaTransactionManager`, `WebSphereUowTransactionManager`
