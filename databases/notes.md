# RDS
- amazon aurora is a different product -> custom db engine created by AWS
- runs within a VPC 
subnet groups: list of subnets that RDS can use
- adding HA will create a primary and standy instance in different availability zones 
- RDS instances have their own dedicated storage from EBS 
- primarys replicate to the secondary syncronously 
- read replicas use async replication and the replicas can go across regions 

Cost of RDS: 

cost 1 -> instance size and type 
cost 2 -> multi AZ or not 
cost 3 -> storage type and amount 
cost 4 -> data transferred 
cost 5 -> backups and snapshots 

# Multi AZ instance and cluster 

multi az instance deployment -> primary instance is synchronously replicating to another AZ standby instance 
  - this is not part of the free tier 
  - one standby replica ONLY 
  - cannot be used for reads or writes -> needs a failover event 
  - only can be within the same region 

multi az cluster deployment -> RDS is capable of having one writer, replicate to two reader instances in different AZs
  - two readers ONLY 
  - readers are usable, they can be used for only read operations
  - you can use this to scale your read workloads
  - endpoint types:
    - cluster endpoint: points at the writer, can be used for read and write 
    - reader endpoint: points at any available reader ( including the writer )
    - instance endpoints: generally used for testing and fault finding
  - runs on much faster hardware -> uses local nvme SSD storage
  - writes are viewed as committed when they're committed to 1 reader 

# RDS backups and restores 

- manual snapshots: store in s3, these are not automated.
  - first snap is full and then incremental after that 
  - if you're using single AZ, this can have performance impact

- automated backups: occur once per day, but they follow the same architecture
  - do this during periods of little use when using single AZ 
  - transactions logs are written every 5 minutes, this means that we have a 5 minute RPO 
  - retention period from 0-35 days

- cross region backups are available 
  - charges apply for the cross region data copy 

Restores
- creates a new RDS instance with a new address
- snapshots -> single point in time 
- automated -> any 5 minute point in time 
- restores aren't fast 

# RDS read replicas
- read only replicas of an RDS instance 
- they aren't part of the main database at all so apps need to be configured to use them 
- asynchronous replication -> data is written to the primary first,  once it's committed, it is replicated to the read replicas
- cross region read replicas -> AWS handles all the networking and in-transit data is encrypted 

performance improvements 
- 5x direct read-replicas per DB instance
- read-replicas can have read-replicas but lag starts to become an issue but, that's because the replication is async
- offer a near 0 RPO
- RR's can be promoted quickly so they have a low RTO
- read only until they're promoted 

# RDS data security
- SSL/TLS ( in transit ) is available for RDS, can be mandatory 
- RDS supports EBS volume encryption with KMS 
- data keys are used for encryption operations, and encryption cannot be removed once configured

- MSSQL and oracle support TDE -> transparent data encryption 
- encryption is handled within the DB engine itself, not the host 

RDS IAM authentication 
- RDS will create a local DB account that is configured to use AWS authentication tokens 
- you create and IAM user and an EC2 role  + IAM policy that maps to the local DB user and allows generate-db-auth-token

Authorization is controlled by the DB engine, this is only allowed for authentication

# AWS Aurora
- uses a cluster with a single primary instance + 0 or more replicas 
- the replicas can be used for failover, as well as read replicas which is different than the baseline RDS configuration
- aurora uses a shared cluster volume for storage, this allows for faster provisioning, improved availability, and performance 

- aurora uses 6 storage nodes, and data is synchronously replicated across all 6 storage nodes
- by default, only the primary can write to the storage but it's available for reading by all the replicas 
- aurora repairs failed disks with the data inside the other storage volumes without corruption 
- up to 15 replicas that you can choose to fail over to
- all SSD based storage 
- you don't specify the amount of storage, it's based on the usage
- replicas can be added without the need for storage configuration

cluster endpoints:
- cluster endpoint: read / write operations 
- reader endpoint: load balanced across all read replicas 
- custom endpoint:

cost:
- high water mark, if you use 50GB of storage and then free up 10GB, you're still billed for the 50Gb
- if you free up all that space and need to reduce your cost, you will need to create a new cluster and migrate the data to that cluster
- no free tier
- hourly charge, per second with 10 minute minimum
- GB month consumed, IO cost per request 
- 100% DB size in backups are included 

features: 
- backups in aurora work the same as RDS 
- restores create a new cluster 
- fast clones: allow you to make a new DB from the new database, references the original storage and and then only stores the new data 
backtrack -> must be configured, allows you to in-place rewind the DB to a previous time 


# Aurora serverless 
- uses ACU -> aurora capacity unites
- set a minimum and max ACU, and that will allow you scale your cluster
- can go to 0 and the DB will be paused 
- consumption is bazsed on billing per second

proxy fleet: used to broker connections between the ACU and the applications
  - apps never directly connect to the ACU, you're connecting to the proxy fleet

use cases:
- infrequently used applications
- new greenfield applications where you're unsure of load 
- variable workloads
- unpredictable workloads
- development and test databases

# Aurora multi master
- default mode is single master -> one DB that can do read and write 
- cluster endpoint is used for reading and writing, read endpoint is used for load balanced reads
- in multi-master mode, all instances are read / write 

- no cluster endpoint being used, apps are responsible for initiating connections to specific instances

how does it work:
- data is proposed for write by a node to all of the storage endpoints
- each of the nodes will either reject of confirm the proposed change
- it is rejected if theres a conflict
- there will need to be a quorum of nodes to write to storage
- data will be written to all the storage nodes
- data is replicated to the in-memory cache of all the other multi master nodes as well 

why is this different than aurora single master: 
- with single master, you connect to a cluster endpoint which if there's failure on the primary will take time to failover, and there will be interruption to the application
- with mutli master, you need to configure connection to both endpoints, and then you can have seamless failover because there isn't any node promotion that needs to occur


# RDS proxy 

why do you want to use RDS proxy?
- opening + closing DB connections consume resources, which adds latency 

- your application will instead connect to the RDS proxy which managed your connections 
- runs only from within a VPC 
- the proxy creates a long term connection pools which allow clients to utilize for database operations 
- connections can be reused..avoiding the lag of establishing and terminating connects for each invocation
- can enforce SSL/TLS connections 

this is very useful for serverless applications -> lambdas will have a higher execution time because they're going to add latency 

# RDS custom
- middleground between RDS and EC2 running DB engine... remember that RDS gives you 0 access to the underlying hardware
- works for MSSQL or oracle 
- can connect using SSH, RDP, and session manager
- customization if performed with database automations 

# DynamoDB 
- noSQL offering 
- key/value could be dynamodb 
Tables -> grouping of items that share the same primary key
- partition keys must be unique 
- max 400KB of size 

primary key types: 
- simple: just uses a partition key
- composite: this uses the partition key and the sort key

capacity in dynamodb:
- this means performance -> we are setting the speed that get used in dyanmodb 
- provisioned: you must set the capacity values 

WCU -> write capacity units 
  - 1 WCU = write 1KB per second
RCU -> read capacity units
  - 1 RCU = read 4Kb per second

backups: 
   - on demand backups: can be used for same region or cross region 
    - allows for adjusting encryption settings
   - Point in time recovery:
      - created continuous stream of backups
      - uses a one second granularity

# dynamodb operations + consistency + performance 

reading and writing 
- on-demand: unknown or unpredictable usage
  - price per million R or W units 
- provisioned: set the RCU and WCU set on a per table basis 
  - every operation consumes atleast 1 RCU/WCU

- every table has a RCU and WCU burst pool ( 300 seconds )

query 
- start with the partition key and optionally an SK or a range

scan:
- least efficient operation
- most flexible operation
- scan goes through a table item by item, so for big tables this can be very costly 
- use this when you are looking for values but you don't know the partition key

# dynamodb consistency module 

- dynamodb data is replicated to multiple AZs by default, they are replicated between storage nodes
  - one of the AZs is the leader of the storage nodes
  - writes are directed to the leader node, this node is known as consistent 
  - the leader will now begin replicated to the reader storage node, if you were to freeze time then you wouldn't have consistency because the third availability zone doesn't have the changes 

types of reads in dynamo
- eventually consistent -> these are half the price as strongly consistent reads, with this though you could be unlucky enough to receive a node that hasn't reached consistency yet
- strongly consistent -> always uses the leader node, so it's the normal cost, and it doesn't scale as well 

WCU calculation
1. calculating WCU per item: (Item size / 1KD)(3)
2. multiply by average number per second (30)

RCU calculation
1. (Item size / 4KB)(1)
2. multiply average read ops per second (10)
3. that gives you the strongly consistent -> to get eventual just divide by 2

# DynamoDB indexes 
- querying is limited in dynamodb because we are only able to work on 1 PK value and optionally a single or range of SK values
what are indexes? -> methods for improving the efficiency for retrieving data in dynamodb 
  - indexed are an alternative view on the table 

Local secondary index -> this is a different SK 
  - you cannot add LSI after the table has been created
  - 5 LSI per base table 
  - alternative sort key on the table, but the same partition key
  - uses the same RCU and WCU on the table

Global secondary index -> this is a different PK and SK 
  - these can be created at anytime 
  - 20 GSI per base table 
  - let you define new PK and SK 
  - they have their own RCU and WCU 
  - always eventually consistent 

# Dynamodb streams and triggers

streams
- time ordered list of item changes in a table 
- 24 hour rolling window
- enable streams on a per table basis 
- records INSERTS, UPDATES, and DELETES

stream view types 
- KEYS_ONLY: only PK and SK are recorded for what is changing 
- NEW_IMAGE: stores the entire item after it has been changed, this cant show what has changed 
- OLD_IMAGE: this shows you what the item looked like before it was changed
- NEW_AND_OLD_IMAGES: pre and post state of the item is recorded

Trigger concepts
- actions to take place once data is actually changed
- item changes generate events, which contains the changed data, and then an action is taken using that data, you do this with lambda functions
- good for reporting and analytics, aggregation and notifications 

Dynamodb Accelerator (DAX) -> create a POC on dax with some timing 
- in-memory cache for dynamodb to improve the performance 

traditional caches vs DAX

generic in-memory cache:
1. app checks cache for data
2. data is missed and gets from the database, and then loads that data into the cache

DAX:
- you need to install the DAX SDK in your application
- when using the DAX sdk, if DAX has your data, it will return it, but it will also go and get the data and hydrate the cache if DAX doesn't get a cache hit

DAX architecture
- dax runs in a cluster architecture with endpoints spread across different availability zones 
- primary node is write and the replicas are read 
- it has an item cache, and a query cache
- cache HITS are returned in microseconds, MISSES are in milliseconds
- write through is supported, writing data to the cache and then to DDB
- faster read operations with a lot less cost
- DAX is deployed within a VPC 

# Dynamodb Global tables
- multi master cross region replication
- tables are created in multiple regions and added to the same global table 
- last writer wins is for conflict resolution 
  - if you have two tables writing to the same record, the most recent write overrides everything else 
- strongly consistent reads only in the same region as writes 

# Dynamodb TTL 
- allows you to set a timestamp that will automatically delete items inside a table 
- you configure the specific attirbute selected for TTL 
- they have no charge and performance impact
- you can create a stream to track the TTL deletions for 24 hours

# AWS elasticsearch 
- managed implementation of elasticsearch
- open source search product which is apart of the ELK stack 
- not serverless, there are servers injected into your VPC 

- elasticsearch - search and indexing services 
- Kibana - visualization and dashboarding tools
- logstash - similar to CWLogs, needs a logstash agnt installed on anything to ingest data 

# Athena
- serverless interactive querying service
- good where loading / transformation isn't desired
- ad-hoc queries on data -> pay only for what you consume 
- schema-on-read - table like transition
- schema translates data -> relational like when read 
- source data is stored on S3, THIS IS NOT MUTABLE DATA
- you use a schema to say what the table is supposed to look like, this isn't enforced until the queries are ran

# Amazon Neptune 
managed graph database service 
- DBs where relationships are as important as data 
- runs within a VPC 
- multiAZ and scales via read replicas 
- continuous backup to s3 // point in time recovery as well 
- like RDS for graph DBs

graph databases are useful for looking through different relationships 

use cases:
- graph style data 
- social media .... anything involving fluid relationships 
- fraud prevention 
- recommendation engines 
- network and IT operations 
- biology and life sciences 

# amazon quantum ledger database
- part of the AWS blockchain products 
- immutable append only ledger based database 
  - crypto verifiable transaction log 
  - transparent -> full history is always accessible 
  - 3AZ resilience and replication between the AZs 
  - can stream data into kinesis 
  - document DB model
  - ACID 

use cases: 
- Finance 
- Medical - full history of data changed 
- logistics
- legal