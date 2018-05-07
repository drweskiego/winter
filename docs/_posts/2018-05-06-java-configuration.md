---
layout: post
title: Container & Dependency Injection
---
## Example configuration

{% highlight java %}
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
{% endhighlight %}

