---
layout: default
---

## Virtual Private Cloud

Virtual Private Clouds enables to launch AWS resources in a virtual network which resembles a traditional network, but with the benefits of a scalable infrastructure. VPC is the networking layer for EC2 and spans all the Availability Zones for a Region.

*Subnets* - A range of IP addresses in a VPC that AWS resources can be launched into. Subnets allow to partition the network within a VPC, and one or more subnets can be added in each AZ.
  * A *public* subnet is a subnet that is accessible from the internet
  * A *private* subnet is not accessible from the internet
  * Each subnet must reside entirely within one AZ and cannot span zones
  * A subnet IP address range is a subset of the VPC CIDR block (e.g. VPC CIDR: 10.0.0.0/16, Subnet 1: 10.0.1.0/24, Subnet 2: 10.0.2.0/24)

*Route Tables* - A set of rules (routes) that determine where network traffic from a VPC should be directed. A subnet can be explicitly associated with a route table, but by default subnets are implicitly associated with the main route table.

The default VPC includes an *internet gateway* which allows instances to access the internet through the Amazon EC2 network edge.

*Network ACL* - An optional layer of security that acts as a firewall which controls inbound and outbound traffic of one or more subnets. NACLs act similarly to Security Groups, but process traffic before it hits any SGs.
  * NACLs operate at the subnet level, whereas SGs operate at the instance level
  * NACLs support allow and deny rules, whereas SGs only support allow rules
  * Are stateless - Return traffic must be explicitely allowed
  * Rules are process in order, starting with the lowest numbered rule

*VPC Flow Logs* - Captures IP traffic information going to and from a VPC, subnets or individual network interfaces (Elastic Network Interfaces). Data is published to CloudWatch Logs or Amazon S3.

*VPC Peering* - VPC Peering is used to privately connect two different VPC networks together, provided they do not have overlapping CIDR address ranges. Instances in either VPC can communicate as if they were within the same network. VPC peering connection is not transitive; i.e. VPC A is connected to VPC B and VPC C. VPC B and C cannot communicate through VPC A; they will need a peering connection between them.

*VPC Endpoints* - Enables private connections between a VPC and supported AWS services. Traffic between a VPC and the other service does not leave the Amazon network. Endpoints do not require an internet gateway, virtual private gateway, NAT device, VPN connection or AWS Directo Connect, and instances do not require public IP addresses. They give enhanced security and lower latency to AWS services.

**NAT**

A NAT device can be used to allow instances in a private subnet to access the internet or other AWS services, but prevent inbound internet connections to the instance.

AWS offers two types of NAT devices:
  1. NAT Gateway - An AWS managed service which provides better availability and bandwidth over NAT instances. NAT Gateways are not a free service. This is the recommended device.
  2. NAT Instance - Launched using a NAT AMI and is not a managed service. Cost is determined by the instance quota.