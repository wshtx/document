# Hibernate Search Guides

>[hibernate-search-orm-elasticsearch](https://quarkus.io/guides/hibernate-search-orm-elasticsearch)

With this guide, you’ll learn how to synchronize your entities to an Elasticsearch or OpenSearch cluster in a heartbeat with Hibernate Search. We will also explore how you can query your Elasticsearch or OpenSearch cluster using the Hibernate Search API.

## Configure ES in a Hibernate Project

`@Indexed`, `@FullTextField`, `@KeywordField`, `@IndexedEmbedded`

```java
package org.acme.hibernate.search.elasticsearch.model;

import java.util.List;
import java.util.Objects;

import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.OneToMany;

import org.hibernate.search.engine.backend.types.Sortable;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.FullTextField;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.Indexed;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.IndexedEmbedded;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.KeywordField;

import io.quarkus.hibernate.orm.panache.PanacheEntity;

@Entity
@Indexed
public class Author extends PanacheEntity {

    @FullTextField(analyzer = "name") 
    @KeywordField(name = "firstName_sort", sortable = Sortable.YES, normalizer = "sort") 
    public String firstName;

    @FullTextField(analyzer = "name")
    @KeywordField(name = "lastName_sort", sortable = Sortable.YES, normalizer = "sort")
    public String lastName;

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.EAGER)
    @IndexedEmbedded 
    public List<Book> books;

    // Preexisting equals()/hashCode() methods
}
```

### [Setting up the analyzers/normalizers](https://quarkus.io/guides/hibernate-search-orm-elasticsearch#analysis-configurer)

It is an easy task, we just need to create an implementation of ElasticsearchAnalysisConfigurer (and configure Quarkus to use it, more on that later).

```java
package org.acme.hibernate.search.elasticsearch.config;

import org.hibernate.search.backend.elasticsearch.analysis.ElasticsearchAnalysisConfigurationContext;
import org.hibernate.search.backend.elasticsearch.analysis.ElasticsearchAnalysisConfigurer;

import javax.enterprise.context.Dependent;
import javax.inject.Named;

@SearchExtension 
public class AnalysisConfigurer implements ElasticsearchAnalysisConfigurer {

    @Override
    public void configure(ElasticsearchAnalysisConfigurationContext context) {
        context.analyzer("name").custom() 
                .tokenizer("standard")
                .tokenFilters("asciifolding", "lowercase");

        context.analyzer("english").custom() 
                .tokenizer("standard")
                .tokenFilters("asciifolding", "lowercase", "porter_stem");

        context.normalizer("sort").custom() 
                .tokenFilters("asciifolding", "lowercase");
    }
}
```

### Adding full text capabilities to our REST service

In our existing `LibraryResource`, we just need to inject the `SearchSession`:

```java
    @Inject
    SearchSession searchSession; 
```

And then we can add the following methods (and a few imports):

```java
    @Transactional 
    void onStart(@Observes StartupEvent ev) throws InterruptedException { 
        // only reindex if we imported some content
        if (Book.count() > 0) {
            searchSession.massIndexer()
                    .startAndWait();
        }
    }

    @GET
    @Path("author/search") 
    @Transactional
    public List<Author> searchAuthors(@RestQuery String pattern, 
            @RestQuery Optional<Integer> size) {
        return searchSession.search(Author.class) 
                .where(f ->
                    pattern == null || pattern.trim().isEmpty() ?
                        f.matchAll() : 
                        f.simpleQueryString()
                                .fields("firstName", "lastName", "books.title").matching(pattern) 
                )
                .sort(f -> f.field("lastName_sort").then().field("firstName_sort")) 
                .fetchHits(size.orElse(20)); 
    }
```

### Configuring the application

>[Hibernate-elasticSearch-configuration](https://quarkus.io/guides/hibernate-search-orm-elasticsearch#configuration-reference)
