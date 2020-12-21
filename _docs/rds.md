---
layout: default
---

#### AWS Relational Database Service (RDS)
Amazon database cloud service which provides cost-efficient and resizable capacity while automating administrative tasks like hardware provisioning, db setup, patching and backups.
  * Available on several instance types
  * Provides six DB engines – Amazon Aurora, Postgre, MySQL, MariaDB, Oracle and Microsoft SQL
  * Managed by Amazon; automated patching, continuous backups, maintenance windows for upgrades, scaling capability (vertical or horizontal)
  * Storage is backed by EBS (gp2 or io1)
  * Cannot SSH into instances

**RDS Backups**
  * Automatically enabled
  * Daily full backup of db during maintenance window
  * Transaction logs backed up every 5 mins
  * 7 day retention period by default, can increase to 35 days

*DB Snapshots* - Manually triggered by user and will retain as long as user wants
*Read Replicas* - RDS uses the DB engines’ built in replication functionality to create a special type of DB instance called a read replica from the source instance. Updates made to primary instance are asynchronously copied to the read replica. Allows to scale out to decrease the load on the primary instance for read-heavy DB workloads.
  * Read replicas are read-only
  * Up to 5 replicas within AZ, cross AZ or cross region
    * Network cost when data goes from one AZ to another
  * Replicas can be promoted to their own DB
  * Apps must update connection string to leverage read replicas
  * Use cases for read replicas:
    * Scaling beyond capacity of single DB instance
    * Serving read traffic while primary DB is unavailable; backups or maintenance
    * Business reporting or data warehousing scenarios where prod DB shouldn’t be used
    * Disaster recovery; read replica can be promoted to standalone instance

RDS Multi AZ (Disaster Recovery)
  * SYNC replication to another standby DB instance
  * One DNS name for automatic failover
  * No manual intervention in apps
  * Not used for scaling; only high availability
  * Can setup read replicas as Multi AZ for disaster recovery

**RDS Security**

RDS uses encryption for data protection

  * At rest encryption
    * Can encrypt master & read replicas with AWS KMS (AES 256)
    * Must be defined at launch; if master is not encrypted, read replicas cannot be encrypted
    * Transparent Data Encryption (TDE) available for Oracle and SQL server
    * Snapshots of un-encrypted RDS DBs are un-encrypted, and vice-versa
    * Can copy an un-encrypted snapshot into an encrypted one
  * In-flight encryption
    * SSL certs to encrypt data in flight
    * Can enforce SSL on Postgres and MySQL with certain commands

To encrypt an un-encrypted RDS database:

    1. Create a snapshot of the un-encrypted DB
    2. Copy the snapshot and enable encryption for it
    3. Restore the DB from the encrypted snapshot
    4. Migrate apps to new encrypted DB, and delete the old un-encrypted DB

RDS Network security
  * DBs usually deployed within a private subnet
  * Leverages security groups to control which IP / SG can communicate with RDS
  * No SSH access – underlying instance is managed by AWS

RDS Access Management
  * IAM policies help control who can manage RDS through RDS API (who can create, delete, modify, etc.)
  * Traditional username/password can be used to login to DB
  * For MySQL and Postgres, IAM-based authentication can be used
    * Need authentication token obtained through IAM & RDS API calls; lifetime of 15 minutes
    * IAM Role makes API call to RDS Service to obtain a token. Then, IAM uses token to authenticate with RDS DB over SSL

**Amazon Aurora**

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

