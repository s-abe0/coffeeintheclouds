---
layout: default
---

## Route 53

*TTL (Time to Live)* - A mandatory value for each DNS record which indicates how long the record will last in DNS cache on clients.

*CNAME* - Points a hostname to any other hostname (app.domain.com -> app2.domain2.com
  * Cannot be used for top node of DNS namespace (zone apex); e.g. domain.com
  * CNAME queries are not free

*Alias* - Points a hostname to an AWS resource
  * Works for root (zone apex) and non-root domain
  * Can redirect queries to AWS resources such as S3 buckets, CloudFront, or another record in the same hosted zone
  * Free of charge and has native health checks

#### Routing Policies
*Simple* - Standard DNS records with no special routing such as weighted or latency. Typically for a single resource such as a web server.
  * Cant create multiple records with same name and type
  * Can specify multiple values in same record. All values returned to client
  * No health checks

*Weighted* - Allows to associate multiple resources with single domain name and choose how traffic is routed to each resource. Can be useful for load balancing or testing new software.
  * Route 53 sends traffic to resource based on weight assigned to the record as a proportion of the total weight for all records in the group:

    ![route53-weighted](/assets/img/route53-weighted.png)

*Latency* - For apps hosted in multiple regions, allows to improve performance for users by serving requests from the region that provides lowest latency.
  * Although a hosted region may be closer to a client, that doesn’t determine the closer hosted region will be the returned IP. Latency for a farther region may be lower than a closer region.

*Failover* - Allows to route traffic to different resource if primary resource becomes unhealthy. Requires a health check to associate with it.

*Geolocation* - Allows to choose resources that server traffic based on geographic location of users.
  * Can be used to localize content and present different languages, to restrict distribution of content to specific regions, or balancing load.
  * Can set a default record that handles locations from unspecified regions. If no default record is created, Route 53 returns a ‘no answer’ response.

  


