---
layout: default
---

## AWS Relational Database Service (RDS)
Amazon database cloud service which provides cost-efficient and resizable capacity while automating administrative tasks like hardware provisioning, DB setup, patching and backups. RDS is available on several database instance types which are optimized for memory, performance or I/O, and provides six popular DB engine types such as Postgres, MySQL, Oracle DB, etc.

Advantages of using RDS over deploying a DB on EC2 instance is that RDS is a managed service, meaning all the basic maintanence tasks are taken care of by AWS. However, SSH access to the RDS instance is not supported. Managed services include:
  * Automated provisioning and OS patching
  * Continuous backups and restore (Point in Time Restore)
  * Monitoring dashboards
  * Read replicas for improved read performance
  * Multi AZ setup for Disaster Recovery
  * Maintanence windows for upgrades
  * Scaling capability
  * Storage backed by EBS (gp2 or io1)

**RDS Backups**

RDS full database backups are automatically enabled and are ran daily during the maintanence window. Transaction logs are backed up every 5 minutes. With these two backups, RDS has the ability to restore to any point in time from the oldest backup to 5 minutes ago. These backups are retained for 7 days, and can be increased to 35 days.

*DB Snapshots* - Backups that are manually triggered by the user, and are retained as long as needed. 

**RDS Read Replicas**

RDS uses the DB engines built-in replication functionality to create read replicas, up to 5. RDS Read Replicas are instances that can be cross AZ or cross Region, and are replicas of the master RDS instance which allow applications to read from, taking load off of the primary instance. Replicas can also be promoted to their own DB instance. Replicas are updated asynchronously, so are eventually consistent. Application connection string needs to be updated to read from read replicas (no one point entry for all replicas)

Use case scenarios:
  * Scaling beyond capacity of a single DB instance for read-heavy workloads.
  * Allow applications to continue to read data while the primary instance is unavailable; such as for maintenence or backups. Read replica data may become 'stale' in this scenario
  * Reporting applications can use a read replica to read from so they don't slow down primary production DB
  * Implementing disaster recovery; read replicas can be promoted to standalone instances.

**RDS Multi-AZ**

RDS Multi-AZ provides high availability and failover support for DB instances. In a Multi-AZ deployment, a synchronous standy replica is provisioned and maintained by AWS in a different AZ. The standby replica cannot be read from, and is only used in the case of the primary instance becoming unavailable during instance failure or AZ distruption.

There is one DNS name for both instances (primary and standby), so no application intervention (connection strings) is necessary.

Read replicas can be set up as Multi AZ.

The primary DB instance switches over to the standby replica during any of the following:
  * An AZ outage
  * The primary instance fails
  * The DB instances server type is changed
  * The OS is undergoing software patching
  * Manual failover of DB initiated using *Reboot with failover*


**RDS Security - Encryption**

*At rest encryption* - Encryption for an RDS instance can be enabled to encrypt data at rest, which includes underlying storage data, automated backups, read replicas and snapshots. AES-256 is used for encryption. 

RDS also supports *Transparent Data Encryption (TDE)* for Oracle and MS SQL Server instances. 

Encryption must be defined at launch time, and if the master is not encrypted, the read replicas cannot be encrypted. However, an un-encrypted DB can be encrypted by performing the following:
  1. Create a snapshot of the un-encrypted DB
  2. Copy the snapshot and enable encryption for it
  3. Restore the DB from the encrypted snapshot
  4. Migrate apps to new encrypted DB, and delete the old un-encrypted DB

*In flight encryption* - SSL or TLS can be used to encrypt connections to a DB instances. Each DB engine provides its own way for implementing SSL/TLS. 

**RDS Security - Network**

RDS databases can be deployed either publicly or privately (within an VPC). Normally, instances are deployed privately, having no public IP or DNS. Instances are secured using security groups, in the same manner as EC2 instances

IAM can be utilized to provide access management; creating IAM policies to control who can manage RDS. IAM database authentication can be used with MySQL and PostgreSQL, which removes the use of passwords when connecting to the DB. Instead of having to store credentials in the database, authentication tokens are used. An *authentication token* is a unique string of characters that RDS generates on request, with a lifetime of 15 minutes.

IAM authentication provides the following benefits:
  * Encrypted network traffic using SSL or TLS
  * Ability to centrally manage access to database instances
  * For apps on EC2, can use profile credentials specific to EC2 instances for DB access, instead of password


#### Amazon Aurora

A fully managed proprietary relational database engine compatible with MySQL and PostgreSQL, allowing existing MySQL and Postgres application to easily migrate to Aurora. It can deliver up to 5x througput of MySQL and 3x throughput of Postgres, and automates clustering, replication and storage allocation. The underlying storage grows automatically as needed, up to 128 Tebibytes (TiB).

**Aurora DB Clusters**

Consists of one or more DB instances and a cluster volume that manages data for those DB instances. 

A *cluster volume* is a virtual DB storage volume that spans multiple AZs, with each AZ having a copy of the cluster data. There are 6 copies of data across 3 AZs; only 4 copies are needed for writes and 3 copies are needed for reads. Self-healling with peer-to-peer replication is provided, and storage is striped across 100s of volumes. 

There are two types of DB instances in an Aurora DB cluster:

*Primary DB instance* - Supports read and write operations, performing all data modifications to the cluster volume. Only one primary instance per DB cluster.

*Aurora Replica* - Supports only read operations from the same storage volume as the primary instance. Each cluster can have up to 15 replicas in addition to the primary instance, spread across separate AZs for high availability. Automatic failover to a replica occurs if the primary instances becomes unavailable.


**Aurora Serverless**

An on-demand, auto-scaling config for Aurora where the DB will automatically start up, scale and shut down based on application needs. It provides a simple, cost effective option for infrequent, intermittent or unpredictable workloads.

*Advantages of Aurora Serverless*

* Simpler - Serverless removes much of the complexity of managing DB instances and capacity. Simply create DB endpoint, optionally specify desired capacity, and connect applications.
* Scalable - Seemlessly scales compute and memory capacity as needed.
* Cost effective - Pay only for resources used on a per-second basis.
* Highly available storage - Uses the same fault-tolerant, distributed storage system as a normal Aurora set-up.


*Use Cases*

* Infrequently used apps - Apps only used for short periods of time per day or week, such as a low volume blog site.
* New apps - When deploying a new application and are unsure about instance size needed.
* Variable workloads - Applications that are lightly used, with peaks of 30 minutes to a few hours a day. Examples are human resources, budgeting and operation reporting apps. 
* Unpredictable workloads - Daily workloads with sudden and unpredictable increases in activity. Example is a traffic site that sees a surge of activity when it starts raining.
* Dev and test databases - Databases used during work hours.

