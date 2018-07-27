---
layout: post
title: Spring Data
---
# Spring Data

Instant repositories

Reposotory is always defined as an **interface** extending `Repository<T, K>` only.

Spring implements interface and creates instances as Spring Beans (using JavaProxy) during runtime

- CRUD methods auto-generated
- Paging, custom queries and sorting supported

Repositories can be implemented for entities annotated with normal JPA annotation (`@Entity`, `@Id` and so on)

Spring provides annotations for other data stores
 - similar annotations to JPA
 - `@Document` map a JSON document (MongoDB)
 - `@NodeEntity`, `@GraphId` map a graph (Neo4J)
 - `@Region` map to a region (Gemfire)

Spring Boot automatically scans for repository interfaces

- Starts in package of `@SpringBootApplication` class and scans all subpackages
- overridden by annotation on `@Configuration` class: `@EnableJpaRepositories(basePackages="com.acme.repository")`


Spring auto-generates implementation for name convention `find(First)By<DataMember><Op>`

- `(First)` - optional
- `<DataMember>` - class/entity field name
- `<Op>` - operation data-member against method args,
  - `GreaterThan`, `NotEquals`, `Between`, `Like`, `IgnoreCase`, ...
  - No `<Op>` for Equals - this is default



 ```java
 // somwere in Spring marker interface defined
public interface Repository<T, ID> { }
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
    public <S extends T> save(S entity);
    public <S extends T> Iterable<S> save(Iterable<S> entities);

    public T findOne(ID id);
    public Iterable<T> findAll();

    public void delete(ID id);
    public void delete(T entity);
    public void deleteAll();
}

public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {

    <S extends T> S saveAndFlush(S entity);
    void flush();

    // Implemented as a single DELETE
    void deleteInBatch(Iterable<T> entities);
    void deleteAllInBatch();

    // Returns a lazy-loading proxy, using JPA's EntityManager.getReference()
    // – equivalent to Hibernate's Session.load()
    T getOne(ID id);
}

/********** consumer code **************/
public interface CustomerRepository extends CrudRepository<Customer, Long> {

    // return null if not found
    public Customer findFirstByEmail(String someEmail);
    public List<Customer> findByOrderDateLessThan(Date someDate);
    public List<Customer> findByOrderDateBetween(Date d1, Date d2);

    // query language dependent on underlying product (here JPQL)
    @Query("SELECT c FROM Customer c WHERE c.email NOT LIKE '%@%'")
    public List<Customer> findInvalidEmails();

    // empty result properly wrapped into Optional
    Optional<Customer> findFirstByEmailIgnoreCase(String email); // Case insensitive search

    @Query("select u from Customer u where u.emailAddress = ?1")
    List<Customer> findByEmail(String email); // ?1 replaced by method param }
}
 ```

 
