# Week 7: Managed Database Services

## HarborTech Ticket Summary
HarborTech received client advisory service request TKT-2026-0007 regarding a cafe business operating a transactional MySQL database on an Amazon EC2 instance. While functional, the database's operating model requires excessive staff time for routine guest OS maintenance, database patching, backup execution and verification, and storage planning. HarborTech was tasked with evaluating whether the client should keep the database on self-managed EC2 or migrate to an AWS-managed relational database service.

## Client Impact
Database availability and performance directly affect the cafe's daily transactional sales and customer order processing. Spending excessive engineering hours on routine server administration increases operational costs and elevates the risk of human error during manual patching or backups. Choosing the wrong service or mismanaging migration risks could cause application downtime or data loss.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Amazon RDS: A fully managed relational database service that takes care of the boring stuff like software updates, security patches, backups, and server setup for you.
- Amazon Aurora: A super-fast, cloud-built database engine designed for big businesses that need high performance, massive scale, and top-tier reliability.
- Amazon DynamoDB: A hands-off, serverless NoSQL database that instantly scales up or down to handle heavy traffic with almost zero delay.
- Relational Databases: Traditional databases that organize info into neat tables with rows and columns, keeping everything strictly connected using SQL.
- NoSQL Databases: Flexible databases that don't use rigid tables, making them perfect for fast key-value lookups or storing documents.
- Multi-AZ: A high-availability feature that automatically copies your database to a backup zone in real time so it stays online if something goes down.
- Database Migration Service: A handy tool that helps you move your databases securely and smoothly into the cloud.
- Schema Conversion: The process of changing your database blueprints and code so they work properly when you switch to a different type of database engine.
- Security Groups: Digital security guards for firewalls that control what traffic is allowed to enter or leave your database instances.
- Database Connectivity: The network paths, web addresses, endpoints, and permissions required for your applications to securely talk to and query your database.

## Virtualization Connection
Running a database on an Amazon EC2 instance virtualizes the physical hardware, but the customer remains entirely responsible for managing the guest operating system, storage volumes, software updates, and patch cycles. A managed database service abstracts these infrastructure layers further. AWS handles the underlying compute host, operating system maintenance, engine updates, and automated backups, leaving the customer responsible only for data design, access patterns, application integration, and workload decisions.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Database engine and workload type: Cafe uses a standard MySQL database to process everyday customer sales and manage store inventory.
- Data structure: Information that is organized into structured SQL tables that rely on relationships and consistent transactional rules
- Query or access patterns: The application routinely reads and writes order records, menu items, and customer information using SQL queries.
- Availability requirements: The database must stay online and reliable during cafe business hours to avoid stopping customer transactions.
- Backup and recovery needs: The business needs reliable, automated daily backups so data can be restored quickly if something breaks.
- Database connectivity: The application connects to the database endpoint over network port 3306 using credentials stored in Systems Manager Parameter Store.
- Security group rules: Inbound firewall rules restrict access on port 3306 so only authorized web server instances can connect to the database.
- Migration requirements: Data needs to move cleanly from the local EC2 server into AWS without corrupting records or causing long application downtime.
- Engine compatibility: Both the source and target run MySQL/MariaDB, so the database can be moved directly without altering any of the current table code.
- Schema conversion needs: None required because the source and target use matching relational database engines.
- Current administrative workload: The client team is wasting too much time manually handling OS updates, software patches, backups, and storage checks.
- Cost or scaling considerations: Relying on EC2 wastes paid staff time on routine maintenance, while moving to RDS automates tasks at a reasonable cost for a single cafe.

## AWS Academy Activity: Actions and Results
Command Entry 1
mysqldump --user=root --password='Re:Start!9' \
--databases mom_pop_db --add-drop-database > mompopdb-backup.sql

Result
[ec2-user@ip-10-200-0-198 ~]$ mysqldump --user=root --password='Re:Start!9' \
> --databases mom_pop_db --add-drop-database > mompopdb-backup.sql
[ec2-user@ip-10-200-0-198 ~]$

This proves that a clean, point-in-time snapshot of the original local mom_pop_db database running on the EC2 instance was successfully captured and exported to a backup SQL file before migration.

Command Entry 2
mysql --user=root --password='ReStart19' \
--host=mompopcafedbinstance.cwfpgbajy2fp.us-east-1.rds.amazonaws.com \
< mompopdb-backup.sql

[ec2-user@ip-10-200-0-198 ~]$ mysql --user=root --password='ReStart19' \
> --host=mompopcafedbinstance.cwfpgbajy2fp.us-east-1.rds.amazonaws.com \
> < mompopdb-backup.sql
[ec2-user@ip-10-200-0-198 ~]$

Shows that network firewalls, security groups, endpoints, and administrative credentials were properly configured to allow data to flow smoothly from the EC2 server into the managed Amazon RDS instance.

Command Entry 3
mysql --user=root --password='ReStart19' \
--host=mompopcafedbinstance.cwfpgbajy2fp.us-east-1.rds.amazonaws.com \
mom_pop_db
select * from product;

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 40
Server version: 10.6.27-MariaDB-log managed by [https://aws.amazon.com/rds/](https://aws.amazon.com/rds/)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [mom_pop_db]>

Validates an active, authenticated interactive session connected to the managed Amazon RDS database, confirming the cloud database is online, accessible, and ready to serve application queries.

## Operational Analysis
Evidence shows that the cafe's database relies on a structured, relational setup with tables, SQL queries, and transactions. The main problem isn't that the database is slow or poorly written; rather, it's the heavy daily chore of managing system updates, security patches, and manual backups on an EC2 virtual server. Because the app requires strict data relationships and transaction support, a managed relational database service like Amazon RDS is the best solution, not a NoSQL database.

## Recommendation
HarborTech recommends migrating the cafe's database from Amazon EC2 to Amazon RDS for MySQL Multi-AZ. This keeps their existing database code working without an app rewrite, while letting AWS handle server maintenance, updates, and backups. The tradeoff is that AWS handles the underlying infrastructure, while the client and HarborTech still control the database design, query optimization, and user permissions.

## Escalation Notes
Currently, no emergency issues or blockers require immediate technical escalation. However, before any final production launch or cutover, a supervisor must review and approve the maintenance window schedule, security group rules, and final cost changes.   

## Lessons Learned
Week 7 taught me that shifting to managed databases like Amazon RDS reduces administrative overhead by letting AWS handle routine chores like patching and backups. It highlighted that workload fit is critical, and we should choose a database based on your data structures for example, use relational engines for structured tables and transactions, or NoSQL for flexible key-value lookups. Additionally, we learned that built-in availability features like Multi-AZ keep apps online during hardware failures, thorough migration planning prevents downtime and data loss, and decisions should always be driven by hard evidence rather than service popularity.

## Professional Vocabulary
Define the important Week 7 terms in your own words.
Include terms such as:
- Relational Database: A type of database that stores information in neat, organized tables with rows and columns, keeping everything connected using SQL rules
- NoSQL: A type of database that doesn't use rigid tables, making it great for storing unstructured data or doing fast, simple lookups.
- Amazon RDS: Cloud service provided by AWS that sets up and manages relational databases for you.
- Amazon Aurora: Database built by AWS that is compatible with MySQL and PostgreSQL and designed for heavy workloads.
- Amazon DynamoDB: Serverless NoSQL database created by AWS that instantly scales up or down to handle traffic smoothly.
- Multi-AZ: High-availability feature that automatically copies your database to a secondary zone in real time
- Database Migration Service: AWS tool that helps you move your databases securely and smoothly from an on-premises server or EC2 instance into the cloud.
- Schema Conversion: A process of changing your database blueprints and code structures.
- Managed Service: An IT model where the cloud provider takes over the responsibility of running and maintaining the underlying infrastructure.
- Workload Fit: Matches your application's actual needs and data patterns with the right type of database service.
