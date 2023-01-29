---
title: Spring Boot 3 MongoDB transactions
date: 2023-01-29 22:12:00 +0400
author: pakisan
categories: [Spring Boot, MongoDB]
tags: [transactions]
---

# Spring Boot 3 MongoDB transactions

## Long story short:

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

## What to do:

- Configure `MongoTransactionManager`
- Create replica set
  - From single node
  - From multiple nodes

## Configure `MongoTransactionManager`

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

## Create replica set: from single node

### Docker

1. Run MongoDB in docker
```shell
docker run -d -p 27017:27017 --name mongodb mongo:6.0.4 mongod --replSet myReplicaSet
```

2. Initiate replica set
```shell
docker exec -it mongodb mongosh --eval "rs.initiate({
 _id: \"myReplicaSet\",
 members: [
   {_id: 0, host: \"localhost\"}
 ]
})"
```

3. Test and verify replica set
```shell
docker exec -it mongodb mongosh --eval "rs.status()"
```

4. Update connection parameters in application properties
```properties
spring.data.mongodb.database=reviews
spring.data.mongodb.replica-set-name=myReplicaSet
```

### Docker compose

1. Create docker compose file:

```
version: "3.9"
networks:
  mongodb-network:
    name: "mongodb-network"
    driver: bridge
services:
  mongodb:
    image: "mongo:6.0.4"
    container_name: "mongodb"
    networks:
      - mongodb-network
    hostname: "mongodb"
    ports:
      - "27017:27017"
    restart: "always"
    entrypoint: [ "/usr/bin/mongod", "--bind_ip_all", "--replSet", "myReplicaSet" ]
  mongoinit:
    image: "mongo:6.0.4"
    container_name: "mongodb_replSet_initializer"
    restart: "no"
    depends_on:
      - mongodb
    networks:
      - mongodb-network
    command: >
      mongosh --host mongodb:27017 --eval "rs.initiate({
       _id: \"myReplicaSet\",
       members: [
         {_id: 0, host: \"mongodb\"}
       ]
      })"
```

2. Launch it

```shell
docker compose up
```

3. Test and verify replica set
```shell
docker exec -it mongodb mongosh --eval "rs.status()"
```

4. Update connection parameters in application properties
```properties
spring.data.mongodb.database=reviews
spring.data.mongodb.replica-set-name=myReplicaSet
```

## Create replica set: from multiple nodes

## References

- [Convert a Standalone to a Replica Set](https://www.mongodb.com/docs/manual/tutorial/convert-standalone-to-replica-set/)
- [Deploying a MongoDB Cluster with Docker](https://www.mongodb.com/compatibility/deploying-a-mongodb-cluster-with-docker)
