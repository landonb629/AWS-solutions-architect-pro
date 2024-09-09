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