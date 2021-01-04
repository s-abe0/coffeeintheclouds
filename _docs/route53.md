---
layout: default
---

## Route 53

A highly available and scalable DNS web service with three main functions: domain registration, DNS routing and health checking. It can use public domain names or private domain names (resolved by instances in VPCs). The most common record types are:
  * A: hostname to IPv4
  * AAAA: hostname to IPv6
  * CNAME: hostname to hostname
  * Alias: hostname to AWS resource

#### Concepts and Record Types

*TTL (Time to Live)* - A mandatory value for each DNS record which indicates how long the record will last in DNS cache on clients. Longer TTL reduces Route 53 clients (reduces amount of DNS queries).

*CNAME* - Points a hostname to any other hostname (app.domain.com -> app2.domain2.com)
  * Cannot be used for top node of DNS namespace (zone apex or root domain); e.g. domain.com
  * Can redirect DNS queries to any DNS record
  * CNAME queries are not free

*Alias* - Points a hostname to an AWS resource
  * Works for root (zone apex) and non-root domain
  * Can redirect queries to AWS resources such as S3 buckets, CloudFront, or another record in the same hosted zone
  * Changes to the resource (e.g. IP address change) are automatically recognized
  * Cannot set TTL; default TTL for the resource is used
  * Free of charge and has native health checks

Alias records can only work with the following AWS resources:
  * Amazon API Gateway custom regional API
  * AWS VPN interface endpoint
  * CloudFront
  * Elastic Beanstalk environment
  * ELB load balancer
  * AWS Global Accelerator 
  * AWS S3 bucket configured as a static website
  * Another Route 53 record in the same hosted zone 

#### Routing Policies

*Simple* - Standard DNS records with no special routing such as weighted or latency. Typically for a single resource such as a web server.
  * Cannot create multiple records with same name and type
  * Can specify multiple values in same record, where all values are returned to the client
  * No health checks

*Weighted* - Allows to associate multiple resources with single domain name and choose how traffic is routed to each resource. Can be useful for load balancing or testing new software (e.g. Send 1% of traffic to a new app version).
  * Route 53 sends traffic to resources based on the weight assigned to the record as a proportion of the total weight for all records in the group:

    ![route53-weighted](/assets/img/route53-weighted.png)

*Latency* - For apps hosted in multiple regions, allows to improve performance for users by serving requests from the region that provides lowest latency.
  * Although a hosted region may be closer to a client, that doesn’t determine the closer hosted region will be the returned IP. Latency for a farther region may be lower than a closer region.

*Failover* - Allows to route traffic to a secondary resource if the primary resource becomes unhealthy. Requires a health check to associate with it.

*Geolocation* - Resources are served based on the users geograpical location (Europe, US, Japan, etc.)
  * Can be used to localize content and present different languages, to restrict distribution of content to specific regions, or balancing load.
  * Can set a default record that handles locations from unspecified regions. If no default record is created, Route 53 returns a ‘no answer’ response.

*Multi Value* - Allows to return multiple values in response to DNS queries. Multiple values can be set in any record, but multivalue routing policies allow to associate health checks for each resource, so Route 53 will return only healthy resources. 
  * It is not a substitute for load balancing, but can improve availability.
  * Up to eight healthy records will be given to a DNS query

  


