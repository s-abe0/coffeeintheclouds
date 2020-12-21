---
layout: default
---

## EC2

*Security Groups* - Fundamental of network security; basically, a firewall
  * Region specific
  * Lives outside the EC2 instance
  * Rules can reference other security groups, allowing instances of that security group access

*Elastic IP* - A fixed public IP address for an EC2 instance
  * Can attach to one instance at a time
  * Can rapidly remap the address to another instance if first instance fails
  * Can only have 5 Elastic IPs per account
  * Should avoid using Elastic IP; use random public IP and register DNS to it, or use load balancer

*Elastic Network Interface (ENI)* - Logical component in a VPC that represents a virtual network card. Can be created independently and attach them to EC2 instances on the fly for failover. ENIs are bound to a specific availability zone.

*EC2 User Data* - Allows to run commands at creation of an EC2 instance. Can find this in Advanced options section when creating an instance.

Example EC2 User Data script installing Apache Httpd:
  ```
  #!/bin/bash

  # get admin privileges
  sudo su

  # install httpd (Linux 2 version)
  yum update -y
  yum install -y httpd.x86_64
  systemctl enable --now httpd.service
  echo "Hola Mundo $(hostname -f)" > /var/www/html/index.html
  ```


**Scalability** - Increasing or decreasing resources based on load
  * Vertical scalability - Increase the size of the instance (t2.micro -> t2.large)
    * Common for non-distributed systems (database)
    * RDS and ElastiCache are services that scale vertically
    * Usually a hardware limit to how much can scale
  * Horizontal scalability - Increase the number of instances; distributed systems
    * Common for web and modern apps

**High Availability** - Distributing systems across geographical locations to avoid a total loss of application availability or data
  * Can be passive; e.g. RDS Multi AZ
  * Can be active; horizontal scaling

**Custom AMI**
  * Can create custom AMI with pre-installed packages, providing faster boot times and more customization to fit business needs.
  * AMI's are built for specific AWS Regions; they are not globably available

#### Instance Launch Types
  * On Demand – Short workload, predictable pricing
    * Pay for use – billing per second after first minute
    * Highest cost but no upfront payment
    * Best for short-term and uninterrupted and elastic workloads
  * Reserved – Minimum 1 year; long workload
    * Up to 75% discount compared to On-demand
    * Pay upfront with long term commitment – 1 or 3 years
    * Best for steady state usage apps (database)
    * Convertible reserved – Can change type (t2.large to t2.xlarge)
    * Scheduled Reserved – Can schedule specific times to run
  * Spot instance – Short workloads for cheap; can lose instances
    * Discount up to 90% compared to On-demand
    * Can lose instance at any time if max price is less than current spot price
    * Best for workloads resilient to failure; batch jobs, data analysis, image processing
  * Dedicated Host – Physical dedicated EC2 server for your use
    * Full control of EC2 instance placement
    * Visibility into underlying sockets / physical cores of hardware
    * Allocated for 3 year reservation
    * Useful for Bring Your Own License software or for strong regulatory or compliance needs
  * Dedicated instance – Runs on hardware dedicated to you
    * May share hardware with other instances on same account
    * No control over instance placement (move hardware after stop/start)

#### EC2 Billing
  * Billing is calculated by either hour or second based on size, OS and region
  * Pricing is per instance hour from start to stop, or per second for partial hour
  * If instance is billed per hour (non-Linux), the full hour is billed at start
  * If instance is billed per second, a minimum of 60 seconds is billed at start

#### EC2 Load Balancers
Load balancers use listeners which checks for connection requests from clients, using specified protocol and port, and forwards the requests to a target group.

ELB Health Checks are crucial as they enable load balancers to know if the instances are available. Health checks are done by default every 5 seconds but can be changed. Health check is done on port and route (/health is common); if response is not 200 (OK), instance is unhealthy.

When setting up security groups, the load balancer gets an open security group to allow HTTP/HTTPS traffic from anywere, but the EC2 instances get a security group to allow traffic only from the Load Balancer.

When enabling an Availability Zone for a load balancer, Elastic Load Balancing creates a load balancer node in that AZ and only forwards traffic to targets in that AZ.

Load Balancer can scale but it is not instantaneously. If a massive scale is required, need to contact AWS for a 'warm-up'.

**Load Balancer components**

*Load balancer* - Serves as single point of contant for clients, distributing incoming application traffic across multiple targets (EC2 instances), in multiple AZs. 

*Listener* - Checks for connections requests from clients, using the protocol and port that you configure. Uses rules that are defined which determines how the load balancer routes requests to its registered targets.

*Target Group* - Routes requests to one or more registered targets. Targets can be EC2 instances, ECS tasks, Lambda functions, or private IP addresses. A listener can have more than one Target Group assigned. **Note:** Classic Load Balancers do not use Target Groups; only Application, Network and Gateway load balancers use Target Groups.

![alt text](/assets/img/alb-components.png)

**Load Balancer concepts**

*LB Stickiness* - Enables the client to be redirected to the same instance behind the load balancer. Uses a ‘cookie’ that has an expiration that can be set. Best use is to allow users to keep session data but may bring imbalances to the load.

*Cross Zone* load balancing allows a load balancer to distribute between all set AZs.
  * Classic LB – Disabled by default; no charge if enabled
  * Application LB – Always on; no charge
  * Network LB – Disabled by default; pay if enabled

*Connection Draining* – Time to complete ‘in-flight’ requests while the target group instance is de-registering or unhealthy. Time is between 1 to 3600 seconds, default is 300, can be disabled to 0. Set a low value if your requests are short, high value if requests are long. This is named as 'Connection Draining' in Classic load balancers, and Deregistration Delay in Application and Network load balancers.

*Server Name Indication (SNI)* – Support for multiple TLS/SSL certificates on ALBs, allowing to host multiple HTTPS apps, each with its own TLS certificate, behind a single load balancer. Only works for Application and Network load balancers.

**Load Balancer types**

*Classic* - Old generation of load balancer (version 1). They expose a public DNS rather than an public IP. Classic load balancers do not use target groups, instead instances are regitstered to it directly. Supports Layer 4 & Layer 7 traffic. It is recommended to use newer generation load balancers (version 2), but use a Classic load balancer if:
  * Need support for EC2-Classic

*Application* - Newer generation (version 2) load balancer that works on Layer 7 (HTTP/HTTPS, WebSocket). They expose a public static DNS rather than an public IP, and do not support Elastic IP assignment. Application load balancers are the most common to use. Target group instances don't see outside world client IPs, but instead the IP of the load balancer. Client IPs are within the X-Forwarded-For header. Benefits of Application load balancer:
  * Use of Target Groups allows to load balance to multiple HTTP apps across multiple machines (or on same machine, but multiple containers)
  * Great for micro-services and container-based apps
  * Supports routing:
      * Based on path in URL ( /posts or /users )
      * Based on hostname in URL ( host1.example.com or host2.example.com )
      * Based on Query String, Headers ( /users?id=123&order=false )

*Network* - Newer generation (version 2) load balancer that works on Layer 4 traffic (TCP, UDP, TLS). NLBs have one static IP per AZ and supports assigning Elastic IPs (helpful for whitelisting specific IP). Best used for high performance, as they have low latency and can support millions of requests per second.
  * When using an NLB (Network Load Balancer), instances in the target group will see the traffic as coming from the outside world; NLBs forward the client IP. Therefore, the target group instances security group needs to be set to allow TCP traffic from *anywhere*


#### Auto Scaling Group (ASG)
Goal of ASG is to scale out (add EC2 instances) or scale in (remove EC2 instances) to match an increased load or decreased load. If an instance under an ASG gets terminated (e.g. marked unhealthy) it will be automatically recreated. They can also terminate and replace instances marked unhealthy by a load balancer.

ASGs have the following attributes:
  * A launch config
    * AMI + instance type
    * EC2 User Data
    * EBS Volumes
    * Security groups
    * SSH Key pair
  * Min size / Max size / Initial Capacity
  * Network + subnet info
  * Load balancer info - ASG can automatically register new instances to a load balancer

*Auto scaling alarms* - Watches for changes to indicate when to scale out / scale in
  * Monitors a metric (e.g. Average CPU, network in/out, # of requests), computed for overal ASG instances
  * Can user CloudWatch alarms

*Scaling cooldowns* - Helps prevent ASG from launching or terminating additional instances before effects of previous activities are visible. In addition to default cooldown periods, scaling-specific cooldowns can be created and applied to a simple scaling policy which override the default cooldown. Common use case is when instances are terminated; ASG needs less time to determine additional actions when instances are being terminated.

**ASG Policies**

*Target Tracking* – Simplest policy to set up; allows to select a scaling metric and set a target value (e.g. CPU usage). CloudWatch alarm is created and managed.

*Step / Simple scaling* – Allows to choose scaling metrics and threshold values for CloudWatch alarms that trigger the scaling process. Can also define how ASG should be scaled when a threshold is met.
  * Main difference between step and simple policies is the step adjustments you get with step scaling. In most cases, step scaling is a better choice.
  * Issue with simple scaling is that the policy must wait for scaling activity to complete and cooldown period to expire before responding to additional alarms.
  * Strongly recommended to use Target Tracking to scale a metric like average CPU utilization or Requests from ALB over Step or Simple scaling policy. Metrics that decrease when capacity increases and increase when capacity decreases can be used to proportionally scale out or in the number of instances using target tracking. This helps insure the ASG follows demand curve for apps closely.

*Scheduled scaling* – Allows to set a scaling schedule; e.g. Traffic is high on Wednesday, so scale out every Wednesday.

#### Elastic Block Store (EBS)
A network drive (not physical) that provides block level storage volumes for EC2 instances that can be mounted as devices and can be dynamically changed. EBS is recommended for data that must be quickly accessible and requires long-term persistence. They can be be detached and attached to another instance very quickly.

EBS Volumes are created in AZs and can only be used in that AZ. To move a volume across an AZ, need to snapshot it.

They have provisioned capacity (in GBs and IOPS). Get billed for all provisioned capacity and can increase capacity over time.

**EBS Volume Types**

*General Purpose SSD (gp2 & gp3)* - Provides balance of price and performance; recommended for most workloads. Supports boot volumes. Supports 3 IOPs per GB up to 16000 IOPS, and is burstable to 3000 IOPS. Volume size is 1 GB to 16 TB. Use cases:
  * Low-latency interactive apps
  * Dev and Test environments

*Provisioned IOPS SSD (io2 and io1)* - Provides high performance for mission-critical, low-latency, or high-throughput workloads that require sustained IOPS performance, or more than 16000 IOPS per volume. IOPS min = 100, max = 64000 (nitro), 32000 (other). IOPS to volume ratio is 50:1. Use cases:
  * Critical business applications that require low latency, sustained IOPS performance
  * Large I/O-intensive database workloads

*Throughput Optimized HDD (st1)* - Low-cost HDD designed for frequently access, throughput-intensive workloads. Supports 500GB to 16TB size. Cannot be a boot volume. Use cases:
  * Big data
  * Data warehouses
  * Log processing

*Cold HDD (sc1)* - Lowest-cost HDD designed for less frequently accessed workloads. Better for large volumes of data that is infrequently accessed, or lowest cost is important. Supports size of 125GB to 16TB. Use cases:
  * Throughput-oriented storage for data that is infrequently accessed.
  * Scenarios where lowest storage cost is important

*Previous generation (standard)* – Use cases is with workloads where data is infrequently accessed.


#### EC2 Instance Store
Provides temporary (ephemeral) block-level storage for an instance that is physically attached to the host computer, therefore provides very high IOPS. It is ideal for temporary storage of info that changes frequently; buffers, caches, scratch data, or for data that is replicated across a fleet of instances (load-balanced pool of web servers)

Instance store volumes can only be created during creation of an instance, and cannot be detached and reatacched to another instance. Only certain EC2 launch types have instance stores (c1, c3, d2, i3, etc.)

Block Storage type (just like EBS), and cannot be resized once created.

Data survives a reboot, but is lost if any of the follow occurs:
  * The underlying disk fails
  * The instance stops, hibernates or terminates


#### Elastic File System (EFS)
Provides simple, scalable, fully managed elastic NFS (Network File System) that can be mounted on many EC2 instances across multiple AZs. Scales on demand to petabytes, growing and shrinking as files are added/removed.

Uses NFSv4 to provide traditonal file structure capabilities. EFS can be access by EC2 instances or on-premise servers using AWS Direct Connect or AWS VPN. EFS uses security groups to control access, and is only compatible with Linux. Encryption at rest using KMS is also available.

**EFS Storage Classes**
*Standard* - Used for frequently accessed files.
*Infrequent Access storage class (EFS IA)* - Cost to retrieve files, but lower price to store data. Cost-optimized for files that are not accessed every day. Using EFS Lifecycle, files not access frequently can be moved to EFS IA.

**EFS Performance Modes**
*General purpose (default)* - For normal use; ideal for latency-sensitive use cases such as web-server, CMS, home directories, etc.
*Max I/O* - Can scale to higher levels of aggregate throughput and ops per second with tradeoff of higher latencies. Best for highly parallelized apps and workloads such as big data analysis, media processing, and genmoics analysis.

**EFS Throughput Modes**
*Bursting (default)* - Throughput scales as file system grows. Some workloads can be spiky, driving high throughput for short periods of time; EFS bursts to high throughput levels for a period of time. Can burst to 100MB/s of throughput; if over 1TB, can burst to 100MB/s per TB of data stored. Bursting uses a credit system, similar to t2 bursting credit system.
*Provisioned* - Allows to specify the throughput of a file system independent of stored data size.

  * Two storage classes:
    * Standard – Used for frequently accessed filed
    * Infrequent Access (EFS IA) – Cost to retrieve files, but lower price to store data. Used for less-frequently accessed files.
  * Two performance modes:
    * General purpose – Ideal for latency-sensitive use cases; web server, CMS, etc.
    * Max I/O – Can scale to higher levels of aggregate throughput and ops per second with tradeoff of higher latencies; good for big data, media processing
  * Two throughput modes:
    * Bursting – Throughput scales as size of file system grows
    * Provisioned – Can provision the throughput independent of storage size
  * Can enable EFS Lifecycle Management, which moves files from Standard to EFS IA after N days
  * Uses security group to control access
  * Encryption at rest using KMS
  * Can only be used with Linux; is a POSIX file system








