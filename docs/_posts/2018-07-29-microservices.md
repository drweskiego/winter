---
layout: post
title: Spring Microservices
---
# Microservices in Spring

## Microservices

- Classic monolithic architecture
  - Single big app
  - Single persistence (usually relational DB)
  - Easier to build
  - Harder to scale up
  - Complex and large deployment process
- Microservices architecture
  - Each microservice has its own storage best suited for it (Relational DB, Non-Relation DB, Document Store, ...)
  - Each microservice can be in different language with different frameworks used, best suited for its purpose
  - Each microservice has separate development, testing and deployment
  - Each microservice can be developed by separate team
  - Harder to build
  - Easier to extend and maintain
  - Easier to scale up
  - Easy to deploy or update just one microservice instead of the whole monolith
- One microservice usually consists of several services in service layer and corresponding data store
- Microservices are well suited to run in the cloud
  - Easily manage scaling - number of microservice instances
  - Scaling can be done dynamically
  - Provided load balancing across all the instances
  - Underlying infrastructure agnostic
  - Isolated containers
- Apps deployed to cloud don't necessary have to be microservices
  - Enables to gradually change existing apps to microservice architecture
  - New features of monolith can be added as microservices
  - Existing features of monolith can be one by one refactored to microservices

## Spring Cloud

- Provides building blocks for building Cloud and microservice apps
  - Intelligent routing among multiple instances
  - Service registration and discovery
  - Circuit Breakers
  - Distributed configuration
  - Distributed messaging
- Independent on specific cloud environment - Supports Heroku, AWS, Cloud foundry, but can be used without any cloud
- Built on Spring Boot
- Consist of many sub-projects
  - Spring Cloud Config
  - Spring Cloud Security
  - Spring Cloud Connectors
  - Spring Cloud Data Flow
  - Spring Cloud Bus
  - Spring Cloud for Amazon Web Services
  - Spring Cloud Netflix
  - Spring Cloud Consul
  - ...

## Building Microservices

1. Create Discovery Service
2. Create a Microservice and register it with Discovery Service
3. Clients use Microservice using "smart" RestTemplate, which manages load balancing and automatic service lookup

#### Discovery Service

- Spring Cloud supports two Discovery Services
  - Eureka - by Netflix
  - Consul.io by Hashicorp (authors of Vagrant)
  - Both can be set up and used for service discovery easily in Spring Cloud
  - Can be later easily switched
- On top of regular Spring Cloud and Boot dependencies (spring-cloud-starter, spring-boot-starter-web, spring-cloud-starter-parent for parent pom), dependency for specific Discovery Service server needs to be provided (eg. spring-cloud-starter-eureka-server)
- Annotate `@SpringBootApplication` with `@EnableEurekaServer`
- Provide Eureka config to `application.properties` (or `application.yml`)

```properties
server.port=8761
#Not a client
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF
```

#### Microservice Registration

- Annotate `@SpringBootApplication` with `@EnableDiscoveryClient`
- Each microservice is a spring boot app
- Fill in application.properties with app name and eureka server URL
  - `spring.application.name` is name of the microservice for discovery
  - `eureka.client.serviceUrl.defaultZone` is URL of the Eureka server

#### Microservice Usage

- Annotate client app using microservice (`@SpringBootApplication`) with `@EnableEurekaServer`
- Clients call Microservice using "smart" RestTemplate, which manages load balancing and automatic service lookup
- Inject RestTemplate using `@Autowired` with additional `@LoadBalanced`
- "Ribbon" service by Netflix provides load balancing
- RestService automatically looks up microservice by logical name and provides load-balancing

```java
@Autowired
@LoadBalanced
private RestTemplate restTemplate;
```

Then call specific Microservice with the restTemplate

```java
restTemplate.getForObject("http://persons-microservice-name/persons/{id}", Person.class, id);
```
