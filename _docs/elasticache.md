---
layout: default
---

## ElastiCache

Allows to set up, run and scale in-memory data stores to build data-intensive apps or boost performance of existing databases. Popular for real-time use cases like Caching, Session Stores, Gaming, Geospatial Services, Real-Time Analytics and Queuing. AWS ElastiCache supports Redis and Memcached engines.

*ElastiCache Nodes* - Smallest building block of an ElastiCache deployment which can exist in isolation from or in some relationship to other nodes. They are a fixed-size chunk of secure, network-attached RAM. Each node runs an instance of the engine and version, and has its own DNS name and port. Nodes can be scaled up or down if necessary.

*ElastiCache Clusters* - An ElastiCache cluster is a logical grouping of one or more ElastiCache nodes (or Redis Shards, for a Redis deployment). Data is partitioned across the nodes (or shards).

**ElastiCache Use Cases**

*In-Memory Data Store* - The overall use for ElastiCache is to increase speed performance of an application. ElastiCache is an in-memory key-value store with provides submillisecond latency, providing ultrafast application response times. Caching database query response data is the most common use-case. 

*User Session Store* - Another use-case for ElastiCache is to store user session data, sort of like SSO. When a user signs into an application, the app stores the users session data. If the user navigates to a different part of the application, the app can query the ElastiCache instance to pull the users session data.

**Redis**

ElastiCache Redis provides the following features:
  * Automatic detection of and recovery from cache node failures
  * Multi-AZ for a failed primary cluster to a read replica
  * Redis clusters support data partition across up to 250 shards
  * From Redis versoin 3.2 and later, in transit and at rest encryption is enabled
  * Flexible AZ placement
  * Backup and restore capabilities
  * Data Durability using AOF persistence – Instance can be stopped and started without losing data
  * Integration with other AWS services


*Redis Shards* - A grouping of one to six related nodes. A multiple node shard implements replication by having one read/write primary and 1-5 replica nodes.

*Redis Cluster (or node group)* - A logical grouping of one or more Redis Shards. A Redis (cluster mode disabled) cluster always has one shard, and a Redis (cluster mode enabled) cluster can have 1-250 shards. Data is partitioned across shards in a Redis (cluster mode enabled) cluster.

*Redis Replication* - Implemented by grouping 2 - 6 nodes in a shard. One node is a read/write primary node, while the others are read-only replicas. Each replica maintains a copy of the data from the primary node, using asynchronous replication mechanisms to keep synchronized. Replicas enhance scalability and fault tolerance, and can be placed in different AZs.

Choose Redis if:
  * Need to authenticated users with role-based access control
  * Want to use Redis streams; a log data structure
  * Encryption at rest and in transit
  * Need persistence and/or backup and restore features
  * Need a more complex caching deployment

**Memcached**

ElastiCache Memcached provides the following features:
  * Automatic detection and recovery from cache node failures
  * Auto Discovery - Automatic discovery of nodes within a cluster so no application changes are required when adding or removing nodes
  * Flexible AZ placement
  * Integration with other AWS services
  * Multi-threaded architecture

However, ElastiCache Memcached has the following downfalls in comparison to Redis:
  * It is non-persistent; if the nodes are stopped, data will be lost
  * No backup and restore features

Memcached *Auto Discovery* provides a configuration endpoint DNS entry which contains the CNAME entries for each cache node enpoint. By connection to the configuration endpoint, and application immediately has information about all the nodes in the cluster.

A Memcached cluster is a logical grouping of one or more elasticache nodes. Data is partitioned across the nodes in a cluster. Memcached supports up to 100 nodes per customer for each Region with a cluster having 1-20 nodes. Clusters can be scaled out or in, which repartitions data across the new number of nodes.

Choose Memcached if:
  * Need simplest model possible
  * Need to run large nodes with multiple cores or threads
  * Need ability to scale out/in
  * Need to cache objects


**Caching Strategies**

When deciding whether to cache data, consider the following:
  * Is it safe to use a cached value? - The data could be out of date
  * Is caching effective for that data? - Certain application generate different access patterns; data can be changing quickly, making caching inefficient
  * Is the data structured for caching? - Because caches are key-value stores, data may need to be cached in multiple different formats to be accessed by different attributes

*Lazy caching (lazy population or cache-aside)* - Populate cache only when an object is requested. Most prevalent form of caching and should serve as foundation of any good caching strategy. Lazy caching should be applied anywhere in an app where data is going to be read often, but infrequently written, such as a user profile for a web app.

Lazy caching psuedo-code example:

  ```
  def get_user(user_id):

      # Check the cache
      record = cache.get(user_id)

      if record is None:       

        # Run a DB query       
        record = db.query("select * from users where id = ?",user_id)

        # Populate the cache
        cache.set(user_id, record)

      return record

  user = get_user(17)
  ```

*Write-through* – Cache is updated when database is updated. For example, user updates their profile, the updated profile is also pushed to the cache. Write-through is usually combined with Lazy Loading. Write-through caching helps avoid unnecessary cache misses, as data is never stale. Example use cases are any type of aggregate, such as top 100 leaderboards, top 10 news stories, or recommendations; this data is normally updated by specific background jobs.

  Advantages:
    * Avoids cache misses, which can help app perform better and feel snappier
    * Shifts any app delay to user updating data, which maps better to user expectations
    * Simplifies cache expiration, as cache is always up to date.

  Disadvantages:
    * Cache can be filled with unnecessary objects that arnt being accessed
    * Can result in lots of cache churn if certain records are updated repeatedly
    * When cache node fails, objects will no longer be in the cache

Write-through psuedo-code example:

  ```
  def save_user(user_id, values):

      # Save to DB 
      record = db.query("update users ... where id = ?", user_id, values)

      # Push into cache
      cache.set(user_id, record)

      return record

  user = save_user(17, {"name": "Nate Dogg"})
  ```

*Time-to-live (TTL)* - Time-to-live for data in a cache can range from a few seconds to a few hours or days. Cache expiration can get very complex, but theres a few considerations that can be used:

  * Apply TTL to all cache keys except those being updated by write-through
  * For fast changing data such as comments, leaderboards or activity streams, use a short TTL of a few seconds
  * Use Russian doll caching; nested records are managed with their own cache keys, and the top level resource is a collection of those cache keys
  * When in doubt, delete a cache key; lazy-loading will refresh the key when needed.

*Evictions* - Occurs when memory is full, resulting in the engine selecting keys to delete (evict). The chosen keys are based on the chosen eviction policy (normally Least Recently Used). If too many evictions are heppening, scale up or out to increase max memory.

**Caching Implementation Considerations:** <https://aws.amazon.com/caching/best-practices/>












