# FSx for windows file server

- native fully managed native windows file shares 
- integrates with directory service or self managed AD 
- single or multi AZ mode within a VPC 
- shares are accessed using SMB 
there is a direct integration with AWS workspaces

features:
- deduplication: process that eliminates excessive copies of data and significantly decreases storage capacity requirements
- distributed file system: file share replicated across multiple servers and locations to increase up-time and reduce access issues related to geography (latency and bandwidth)
- KMS at rest encryption 
- enforced encryption in transit

for the exam:
- VSS -> user driver restores ( file and folder level )
- natively accessible over SMB
- windows permission model is used by FSx


# FSx for Lustre

- managed Lustre - designed for HPC - linux clients ( POSIX )
- machine learning, big data, and financial modeling
- scales to 100's GBs throughput and sub milisecond latency 

deployment types 
- scratch:
  - highly optimized for short term, no replication
  - if there is failure to underlying hardware, the data will be gone 
- persistent
  - longer term, HA ( in one AZ ), self healing

How it works:
- data is lazy loaded from s3 into the file system as needed
- the data lives in the file system while the processing occurs
- data will exported back into the s3 bucket after it's been processed

Features and info:
- baseline performance is based on size
- size - minimum 1.2TiB then increments of 2.4TiB
- scratch gives you 200MB/s per TiB of storage 
- peristent offers 50MB/s, 100MB/s, and 200 MB/s per TiB of storage 

you will typically have some kind of compute that has the lustre client installed that allows you to interact with the lustre file system

- lustre runs from one AZ, so you have an ENI in your VPC that you interact with 
- frequently accessed data uses in memory caching
- larger the file system, the higher chance of failure

- Persistent has replication within a single AZ 
  - so if the entire AZ goes down, you could have some data loss

- you can backup to s3 with both scratch and persistent storage

# EFS refresher

- uses network based file systems which can be mounted to EFS 
- uses NFSv4 
- EFS filesystems can be mounted in linux
- network connected to VPCs, so you have all the ability to connect with hybrid architectures

mount targets -> this is how you connect to EFS file system

for exam:
- can only be used with linux 
- two modes: general purpose, and max IO performance 
- throughput modes: bursting and provisioned 
- storage classes: infrequent access, and standard
- lifecycle policies can be used with EFS 

# S3 object storage classes

Standard -> default storage class for s3
- across 3 AZs 
- 11 nines of durability
- cost considerations:
  - GB pepr month fee for data stored
  - cost per GB charge for transfer OUT 
  - cost per 1k requests 

Standard Infrequent Access (Standard-IA):
- across 3 AZs 
- durability is the same
- cost considerations:
  - per GB retrieval fee 
  - minimum duration charge of 30 days, objects can be stored for less, but they're always billed with the minimum 
  - minimum capacity charge of 128KB per object
- use when data access is infrequent, but the data cannot be replaced easily

S3 One Zone Infrequent Access:
- data is only stored in one availability zone
- you get the same durability, as long as the AZ doesn't fail
- cost considerations:
  - One zone IA has a minimum duration charge of 30 days 
  - objects can be stred for less, but minimum billing always applies 
  - minimum capacity of 128KB per object
- use for long lived data that can be easily replaces, and it's non critical ( replica copies )

S3 Glacier - Instant:
- when you need data instantly, but you need it once a quarter
- Cost considerations:
  - minimum duration charge of 90 days
  - objects can be stored for less but minimum billing always applies
  - stored across 3 AZs

S3 Glacier - Flexible retrieval: 
- store archival data where the storage isnt immediately needed
- objects are replicated across 3 AZs
- 1/6th of the cost of s3 standard
- these objects are stored as cold storage, they're not immediately available
- objects cannot be made publicly available
- data retrieval job types:
  - expedited -> 1-5 minutes
  - Standard -> 3-5 hours
  - bulk -> 5-12 hours

Once you retrieve data from this storage tier: 
- objects arwe stored in s3 infrequent access, and then later removed


S3 Glacier deep archive: 
- cheapest level of storage
- data is in a frozen state
- 180 minimum billable duration 
retrieval job type
- standard -> 12 hours 
- bulk -> up to 48 hours

S3 intelligent tiering:
- objects are automatically moved between different tiers 
- has a monitoring and automation cost per 1000 objects
- used for long lived data with changing and unknown data patterns
- storage tiers:
  - frequent access
  - infrequent access
  - archive instant access 
  - archive access ( optional ) -> apps must support async data 
  - deep archive ( optional ) -> apps must support async data 


# S3 lifecycle configuration

- automatically transition or expire objects in S3 
- can apply to entire bucket, or groups of objects by prefix or tags
- transition actions or expiration actions 
- actions cannot be set for access patterns ( that is intelligent tiering )

a single rule cannot transition to standard-IA or One zone IA and then to glacier classes within 30 days ( duration minimums )
- cannot have a rule that moves data from standard to standard-IA or one zone-IA, and then to glacier all within 30 days


# S3 replication 
- Cross region replication -> replicate to one or more buckets in another region 
- same region replication -> replicate to one or more buckets in the same region

- buckets can be in the same or a different AWS ACCOUNT 

replication configuration:
- need to specify the target bucket
- need to specify the IAM role

configuring replication between different accounts:
- role isn't trusted by default by the destination account
- you will need to add a bucket policy on the destination account, to allow the role in the source account

Replication options:
- all objects or a subset of objects, a rule can filter to a prefix or tags 
- storage class -> default is to use  the same class as the source bucket
- ownership -> default is owned by same account as source bucket, if buckets are in different account, then the destination account may not be able to access the objects
- replication time control -> adds guaranteed SLA to replication, without it, it's just best effort for replication

for the exam: 
- if you enable replication on a bucket that already has objects, those will not be replicated 
- versioning must be turned on for replication to work ( source and destination )
- s3 batch replication can be used to move existing objects
- one way replication ( not bi-diretional ), you can make it bi-directional but that is an extra configuration 
- can handle unencrypted, or KMS with extra configuration, SSE-customer manager keys can also be configured
- no system events, glacier or glacier deep archive cannot be replicated
- by default DELETES are not replicated, this can be configured though