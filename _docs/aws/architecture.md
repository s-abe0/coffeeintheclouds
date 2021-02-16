---
layout: default
---

## AWS Architectures

**Three Tier architecture**

Basic setup for different kinds of applications. Fronted by a Load Balancer using Route 53 for DNS, the Load Balancer connects to an Auto Scaling group of EC2 instances spread across multiple AZs. The auto scaling group is deployed within a private subnet in the VPC. Behind the ASG is the Data tier, where an RDS and perhaps an ElastiCache instance are sitting in a Data subnet. 

![alt text](/assets/img/3-tier-arch.png)

**EC2 LAMP Stack**

Linux: OS for EC2 instances
Apache: EC2 web server instance
MySQL: RDS database
PHP: App logic on EC2

Basic setup for any sort of basic web page. ElastiCache Redis or Memcached instance could be use to include caching. Local app data and software can be stored using EBS volumes.