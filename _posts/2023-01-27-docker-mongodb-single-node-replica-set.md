---
title: MongoDB single node replica set with Docker | Step-by-Step Guide
date: 2023-01-27 16:00:00 +0400
author: pakisan
categories: [MongoDB, Replica set]
tags: [transactions, mongodb, docker]
mermaid: true
description: Learn how to set up a MongoDB single node replica set using Docker with this step-by-step guide. Perfect for development environments and testing MongoDB transactions
head:
  - - meta
    - name: keywords
      content: mongodb replica set, docker mongodb, single node replica set, mongodb transactions, docker mongodb setup, mongodb cluster, mongodb replication
  - - meta
    - property: og:title
      content: MongoDB single node replica set with Docker | Step-by-Step Guide
  - - meta
    - property: og:description
      content: Learn how to set up a MongoDB single node replica set using Docker with this step-by-step guide. Perfect for development environments and testing MongoDB transactions
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/docker-mongodb-single-node-replica-set/
  - - meta
    - name: twitter:title
      content: MongoDB single node replica set with Docker | Step-by-Step Guide
  - - meta
    - name: twitter:description
      content: Learn how to set up a MongoDB single node replica set using Docker with this step-by-step guide. Perfect for development environments and testing MongoDB transactions.
---

## Instructions

The steps to create a docker cluster are as follows.
- Start one instance of MongoDB.
- Initiate the Replica Set.

Once you have a MongoDB cluster up and running, you will be able to experiment with it.

How it will look like:

```mermaid
graph LR
    Application[Application] -- Read --> MongoDB(MongoDB)
    Application[Application] -- Write --> MongoDB(MongoDB)
```

### Run MongoDB in docker
```shell
docker run -d -p 27017:27017 --name mongodb mongo:6.0.4 mongod --replSet myReplicaSet
```

### Initiate replica set
```shell
docker exec -it mongodb mongosh --eval "rs.initiate({
 _id: \"myReplicaSet\",
 members: [
   {_id: 0, host: \"localhost\"}
 ]
})"
```

### Test and verify replica set
```shell
docker exec -it mongodb mongosh --eval "rs.status()"
```

## References
- [Convert a Standalone to a Replica Set](https://www.mongodb.com/docs/manual/tutorial/convert-standalone-to-replica-set/)
