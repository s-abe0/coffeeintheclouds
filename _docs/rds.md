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

*At rest encryption* - Encryption for an RDS instance can be enabled to encrypt data at rest, which includes underlying storage data, automated backups, read replicas and snapshots. AES-256 is used for encryption. RDS also supports Transparent Data Encryption (TDE) for Oracle and SQL Server instances. 

Encryption must be defined at launch time, and if the master is not encrypted, the read replicas cannot be encrypted. However, an un-encrypted DB can be encrypted by performing the following:
  1. Create a snapshot of the un-encrypted DB
  2. Copy the snapshot and enable encryption for it
  3. Restore the DB from the encrypted snapshot
  4. Migrate apps to new encrypted DB, and delete the old un-encrypted DB

*In flight encryption* - SSL or TLS can be used to encrypt connections to a DB instances. Each DB engine provides its own way for implementing SSL/TLS. 

**RDS Security - Network**

RDS databases can be deployed either publicly or privately (within an VPC). Normally, instances are deployed privately, having no public IP or DNS. Instances are secured using security groups, in the same manner as EC2 instances

IAM can be utilized to provide access management; creating IAM policies to control who can manage RDS. IAM database authentication can be used with MySQL and PostgreSQL, which removes the use of passwords when connecting to the DB. Instead of having to store credentials in the database, authentication tokens are used. And *authentication token* is a unique string of characters that RDS generates on request, with a lifetime of 15 minutes.

IAM authentication provides the following benefits:
  * Encrypted network traffic using SSL or TLS
  * Ability to centrally manage access to database instances
  * For apps on EC2, can use profile credentials specific to EC2 instances for DB access, instead of password


#### Amazon Aurora

A proprietary technology from AWS which is MySQL and Postgres compatible.
  * Up to 5x faster than standard MySQL DB and 3x faster than standard Postgres DB
  * Provides security, availability and reliability of commercial DB at 1/10th the cost
  * Fully managed by RDS; automates administration tasks (patching, backups, etc)
  * Distributed, fault-tolerant, self-healing storage auto-scaling up to 128TB in increments of 10GB
  * Up to 15 read replicas; replication process is much faster
  * Provides point-in-time recovery, continuous backup to S3, and replication across three AZs
  * Costs more than RDS (20% more), but is more efficient

Aurora High Avaliability
  * Replicates 6 copies of data across 3 AZs, backing up data to S3
    * Only needs 4 copies of 6 needed for writes, 3 copies of 6 needed for reads; If one AZ goes down, your still OK
  * Self-healing – If some data goes bad, uses peer-to-peer replication to recover
  * Storage is striped across 100s of volumes
  * One Aurora instance takes master role; if master fails, read replica can become master

**Aurora DB Cluster**

Consists of one or more DB instances and a cluster volume that manages data for those instances. Cluster volume is a virtual DB storage volume that spans multiple AZs. Two types of DB instances make up an Aurora cluster:
    1.	Primary instance – Supports read/write operations and performs all modifications to cluster volume. One primary per cluster.
    2.	Aurora Replica – Connects to same storage volume as primary and supports only reads. Up to 15 replicas per cluster in addition to primary.

Cluster Endpoints
An Aurora-specific URL which allows connecting to various parts of the Aurora DB cluster
    *Cluster endpoint (writer endpoint)* – Connects to current primary DB instance. Only one that can perform write operations such as DDL (Data Definition Language) statements.
    *Reader endpoint* – Provides load balancing support for read-only connections to the read replicas.

*Aurora Security* – The same as RDS

Aurora Serverless

An on-demand, auto-scaling config for Aurora where the DB will automatically start up, shut down and scale based on application needs. Good for infrequent, intermittent or unpredictable workloads.

Simply create DB endpoint, optionally specify desired capacity, and connect applications. Pay on a per-second basis for capacity used when DB is active. Can migrate between standard and serverless configs.

Use cases
  * Infrequently used apps – App is only used for a few minutes several times a day or week
  * New apps – Deploying a new app and are unsure which instance size you need
  * Variable workloads – Running lightly used app with peaks of 30 minutes to several hours a few times each day or year (human resources, budgeting, reporting app)
  * Unpredictable workloads – Workloads where there is DB usage throughout the day, and peaks of activity that are hard to predict. 
  * Dev and Test databases – DB used during work hours, but will automatically shut down when not in use.
  * Multitenant Apps – Web app with DB for each customer. Aurora manages individual capacity

Aurora Global DB

Designed for globally distributed apps, allowing a single Aurora DB to span multiple regions. Consists of one primary region and up to 5 read-only, secondary regions. Up to 16 read replicas per secondary region. Can promote a secondary region to primary.

