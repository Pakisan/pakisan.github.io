---
title: Spring Boot 3 MongoDB transactions
date: 2023-01-29 22:12:00 +0400
author: pakisan
categories: [Spring Boot, MongoDB]
tags: [transactions, mongodb]
mermaid: true
---

Transactions won't work without next things:
- replica set

> Since transactions are built on concepts of logical sessions they require mechanics (like oplog) which are only available in replica set environment.
>
> You can always convert a standalone to a single noded replica set and transactions will work with this one node.
>
> https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/

[Reference](https://www.mongodb.com/community/forums/t/why-replica-set-is-mandatory-for-transactions-in-mongodb/9533)

- MongoTransactionManager

> Unless you specify a `MongoTransactionManager` within your application context, transaction support is DISABLED. You can use `setSessionSynchronization(ALWAYS)` to participate in ongoing non-native MongoDB transactions.

[Reference](https://docs.spring.io/spring-data/mongodb/docs/4.0.1/reference/html/#mongo.transactions)

That's main reason why newly created Spring Boot project with MongoDB will fail with next error:

```
java.lang.IllegalStateException: Failed to retrieve PlatformTransactionManager for @Transactional test:
```

# What to do:

- Create replica set
- Configure `MongoTransactionManager` as `PlatformTransactionManager`
- Update application properties
- Fix DataMongoTest tests

## Create replica set

Choose preferred way to create replica set:
- [Docker single node replica set](/posts/docker-mongodb-single-node-replica-set)
- [Docker multiple node replica set](/posts/docker-mongodb-multiple-node-replica-set)
- [Docker compose single node replica set](/posts/docker-compose-mongodb-single-node-replica-set/)

## Configure PlatformTransactionManager

Register `MongoTransactionManager` in the application context.

```java
import org.jetbrains.annotations.NotNull;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.MongoDatabaseFactory;
import org.springframework.data.mongodb.MongoTransactionManager;
import org.springframework.data.mongodb.config.AbstractMongoClientConfiguration;

@Configuration
public class MongoClientConfiguration extends AbstractMongoClientConfiguration {

    @Value("${spring.data.mongodb.database}")
    private String databaseName;

    @Bean
    public MongoTransactionManager transactionManager(MongoDatabaseFactory mongoDatabaseFactory) {
        return new MongoTransactionManager(mongoDatabaseFactory);
    }

    @NotNull
    @Override
    protected String getDatabaseName() {
        return databaseName;
    }

}
```

## Update application properties

```properties
spring.data.mongodb.replica-set-name=myReplicaSet
```

## Fix DataMongoTest tests

Looks like `@DataMongoTest` still not activates autoconfiguration for MongoDB transactions.

References:
- [Issue on StackOverflow](https://stackoverflow.com/questions/60178310/spring-data-mongodb-transactional-isnt-working/60184283)
- [Closed issue on GitHub](https://github.com/spring-projects/spring-boot/issues/20182)

Proposal to import `TransactionAutoConfiguration` didn't help me
```java
@ImportAutoConfiguration(TransactionAutoConfiguration.class)
```

So I decided to replace it with MongoDBConfig import
```java
import app.config.MongoDBConfig;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;
import org.springframework.context.annotation.Import;
import org.springframework.transaction.annotation.Transactional;

@DataMongoTest
@Transactional
@Import(MongoDBConfig.class)
public class MongoTest {

    @Test
    public void testTransaction() {

    }

}
```

# References

- [Convert a Standalone to a Replica Set](https://www.mongodb.com/docs/manual/tutorial/convert-standalone-to-replica-set/)
- [Deploying a MongoDB Cluster with Docker](https://www.mongodb.com/compatibility/deploying-a-mongodb-cluster-with-docker)
