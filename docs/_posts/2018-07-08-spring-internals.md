---
layout: post
title: Spring internals
---
# Spring internals

## Simple lifecycle

- Initialization
  - Spring Beans are created
  - Dependency Injection occurs
- Usage
  -Beans are available for use in the application
- Destruction
  -Beans are released for Garbage Collection

## Initialization

- **Load Bean Definition**
  - `@Configuration` classes processed
  - `@Components` annotation scanned for
  - XML files parsed
  - bean definitions added to `BeanFactory` - indexed by id and type
- **Post Process Bean Definition**
  - `BeanFactoryPostProcessor` beans invoced on bean definitions (able to **update definitions**)
    - `void	postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)` method signature
    - declre by `@Component` class annotation or `@Bean` **static** method annotation
    - spring implementations reads properties, registaring scopes...
- for each bean
  - Find and create dependences
  - Instattiate Bean (constructor dependenced injection)
  - Call Setters (injection)
  - **Bean Post Processors**
    - BPP
    - Initializer
    - BPP
- Bean ready

