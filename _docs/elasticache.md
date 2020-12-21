---
layout: default
---

## ElastiCache
Allows to set up, run and scale in-memory data stores to build data-intensive apps or boost performance of existing databases. Popular for real-time use cases like Caching, Session Stores, Gaming, Geospatial Services, Real-Time Analytics and Queuing.

*ElastiCache Nodes* - Smallest building block of an ElastiCache deployment, a fixed-size chunk of secure, network-attached RAM. Each node runs an instance of the engine and version. Can scale nodes up or down if necessary. Each node has its own DNS and port.
  * Helps make apps stateless by storing state
  * Write scaling using sharding
  * Read scaling using read replicas
  * Multi AZ with Failover capability
  * Support for Redis and Memcached
  * Nodes can be pay-as-you-go or reserved

**Solution Architecture**
  * DB Cache - Apps query ElastiCache; if data not available, get from RDS and store in ElastiCache. Cache must have invalidation strategy to make sure data is current.
  * User Session store – Store user session data in ElastiCache instance; user profile, preferences, past shopping history

**Redis**

*Redis Shard* - A grouping of one to six related nodes. A Redis (cluster mode disabled) cluster always has one shard, and a Redis (cluster mode enabled) cluster can have 1-90 shards. A multiple node shard implements replication by having one read/write primary and 1-5 replica nodes.

*Redis Cluster* - A logical grouping of one or more Redis Shards. Data is partitioned across shards in a Redis (cluster mode enabled) cluster.

  * Multi AZ and automatic failover
  * Version 3.2 and later; support for encryption in transit and at rest with authentication
  * Read replicas to scale reads and high availability
  * Data Durability using AOF persistence – Instance can be stopped and started without losing data
  * Backup and restore features

Choose Redis if:
  * Need to authenticated users with role-based access control
  * Want to use Redis streams; a log data structure
  * Encryption at rest and in transit
  * Need persistence and/or backup and restore features

**Memcached**

A Memcached cluster is a logical grouping of one or more elasticache nodes. Data is partitioned across the nodes in a cluster. Memcached supports up to 100 nodes per customer for each Region with a cluster having 1-20 nodes.

  * Multi AZ
  * Can scale out or in by adding or removing nodes
  * Non-persistent 
  * Multi-threaded architecture

Choose Memcached if:
  * Need simplest model possible
  * Need to run large nodes with multiple cores or threads
  * Need ability to scale out/in
  * Need to cache objects

**Caching Implementation Considerations:** <https://aws.amazon.com/caching/best-practices/>

*Lazy caching (lazy population or cache-aside)* - Populate cache only when an object is requested. Most prevalent form of caching and should serve as foundation of any good caching strategy.

*Write-through* – Cache is updated when database is updated. For example, user updates their profile, the updated profile is also pushed to the cache. Write-through is usually combined with Lazy Loading.

  Advantages:
    * Avoids cache misses, which can help app perform better and feel snappier
    * Shifts any app delay to user updating data, which maps better to user expectations
    * Simplifies cache expiration, as cache is always up to date.

  Disadvantages:
    * Cache can be filled with unnecessary objects that arnt being accessed
    * Can result in lots of cache churn if certain records are updated repeatedly
    * When cache node fails, objects will no longer be in the cache






