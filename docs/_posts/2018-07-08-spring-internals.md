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
- **Post Process Bean Definition**
- for each bean
  - Find and create dependences
  - Instattiate Bean (constructor dependenced injection)
  - Call Setters (injection)
  - **Bean Post Processors**
    - BPP
    - Initializer
    - BPP
- Bean ready

