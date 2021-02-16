---
layout: default
---

## AWS Regions and Zones

* Regions all around the world. A region is a cluster of data centers
* Some services are either global (IAM) or region specific (EC2)
* Some services are only available in certain regions
* Names can be us-east-1, eu-west-3, etc.

#### Availability Zones

* Each region as many availability zones; usually 3 (min of 2, max of 6)
* Each availability zone (AZ) is one or more discrete data centers with redundant power, networking and connectivity.
* Separated and isolated from disasters; connected with high bandwidth & low latency
* ap-southeast-2a, ap-southeast-2b, ap-southeast-2c, etc.

![alt text](/assets/img/aws-regions.png)
