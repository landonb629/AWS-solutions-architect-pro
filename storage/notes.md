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
- you cannot have a rule that trasfers from standard to s3-IA or one zone IA within 30 days 
- if an objects is stored on s3 standard, it cannot be transitioned to glacier unless it's a day after


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

# S3 object encryption CSE/SSE
- encryption is configured at the object level 

Client side encryption -> objects being uploaded are already encrypted by the client, AWS cannot see the data in its plaintext form
- use this when you need to manage the keys and encryption + decrytion process

Server side encryption -> objects aren't initially encrypted by s3, in theory if someone was able to listen to the HTTPS upload tunnel, the objects would be in plaintext
- SSE-C: encrytion using customer managed key. you managed the key, s3 managed the encryption. you give the key and the plaintext object, there is a one way hash and the ciphertext object that is stored, you need to give the key to unencrypt the object

- SSE-S3: default encryption managed by s3. you upload the unencrypted object, and aws will manage the key and the encryption of the object. you have little control over the actual key that is used for encryption. AES-256.

- SSE-KMS: encryption using a KMS key. using customer managed KMS key, you have full configuration ability on that key. gives you fine grained control over the key itself. 


# S3 bucket keys

How does encryption work with the default encryption key? 
1. for every put call, aws s3 will call kms to generate a data encryption key that gets stored with the objects, this is an api call to KMS also 

why does this cause problems?
- calls to KMS have a cost & levels where throttling occur

what are bucket keys? 
- there is a time limited bucket key that gets created to generate the data encryption keys within S3, this is lower the amount of calls that need to be made to KMS 

# s3 presigned URLs
- provides access with a URL that has authentication information for interacting with s3 objects 
- typically used in serverless architectures when you want to control access into the s3 bucket
- time limited, and authentication information is baked into the URL 

EXAM SCENARIO:
- you can create a presigned URL for and object that you don't have any access to, you will still not have access though
- permissions on the URL match the identity that generated it
- don't use presigned URL with an assumed role, the URL will stop working once the temporary credentials expire


# S3 select & glacier select 

s3 / glacier select -> allows you to access parts of objects using SQL-like statements, so it's faster and cheaper
- CSV, JSON, parquet files
- since the data is filtered in s3, you get up to 400% speed increase and 20% cheaper

# S3 access points 
- simplifies acess management to s3 buckets / objects
- you can create many access points that have different policies associated with them
- each access point has it's own endpoint address
- allows you to not have to deal with large bucket policies

aws s3control create-access-point

- access points can be assigned specifically to a VPC endpoint 

# S3 object lock
- enable on new s3 buckets -> if you want to do it on existing buckets, you need to request via AWS support
- write-once read-many -> no delete, no overwrite
- requires versioning

These can be defined on each object:
- retention period: specify days & years 
  - compliance mode: cant be adjusted, deleted, or overwritten, cant reduce the retention settings
  - Governance mode: special permissions can be granted allowing the lock settings to be adjusted
    s3:BypassGovernanceRetention is the necessary permission, you also need to pass a header
- legal hold 
  - don't set any retention period, you just set this on or off 
  - no deletes or changes until removed
  - s3:PutObjectLegalHold is required to add or remove

# Amazon Macie
- data security and privacy service
- discover, monitor, and protect data stored in s3 buckets 
- automated discovery of data ie Pii, PHI, finance
- you create discovery jobs in macie and they will search in your configured buckets, they have findings generated
- findings: 
  - policy findings -> actions that would reduce the security of the s3 bucket
  - sensitive data findings -> discovers sensitive data in s3 buckets

Managed data identifiers -> built into the product using ML patterns 
Custom data identifiers -> Regex Based

Integrates with security hub & findings events to eventbridge

Centrally manage in one account -> either via AWS orgs or one macie account which is inviting

# EBS volumes 

General purpose SSD - GP2
- 1GB - 16TB 
- every volume has a baseline performance based on size 
- you receive a baseline of IO credits, and they're refilled with minimum 100 IO credits per second regardless of volume size
- all volumes are credited 3 IO per second for every GB in size up to 1 TB 
- if your volumes are greater than 1TB, credit system isn't used & you always achieve baseline 

General purpose SSD - GP3 
- 3000 IOPS & 125MB/s - standard
- base price is 20% cheaper than GP2 
- there is no credit system with GP3

Provisioned IOPS SSD ( io1 / io2 )
- IOPS can be adjusted independently of size 
- consistent low latency + jitter
- block express -> 
- io1 - 260,000 IOPS & 7500 MB/s max per instance
- io2 - 160,000 IOPS & 4750 mb/s max per instance 
- io2 block express - 260,000 IOPS & 7500 MB/s max per instance
think of these per instance as a performance cap 


HDD based EBS volume types 
- st1 -> throughput optimized, cheaper than SSD
  - 125GB - 16TB 
  - max 500 IOPS
  - baseline performance 40MB/s base 
  - 250MB/s burst
  - use with big data, data warehouses, and log processing
- sc1 -> cold HDD, cheapest
  - for infrequent workloads
  - max 250 IOPS 
  - lowest cost EBS volume


Instance store volumes
- TEMPORARY STORAGE -> if you shut off your EC2, or if the instance moves between hosts, the data is lost
- block storage devices, attached directly to the EC2 instance
- physically connected to one EC2 host, they're not available over the network 
- highest storage performance on AWS 
- included on the price on the instance
- ATTACHED ONLY AT LAUNCH, CANNOT REATTACH MORE

- should be used for buffers, caches, scratch data, and other temporary content


When would you choose EBS vs instance store:
- persistence, or resilience, isolated from instance lifecycle -> use EBS
- super high performance needs, and cost -> instance store volumes

Cheap -> ST1 or SC1 
throughput or streaming -> ST1
Boot -> cannot use ST1 or SC1 

GP2/3 -> up to 16,000 IOPS
IO1/2 -> 64,000 iOPS or 256k with block express 
RAID0 -> up to 260,000 with block express 
Instance store can give you up to millions of IOPS

# AWS Transfer Family 
- Managed file transfer service - supports TO or FROM s3 and EFS 
- provides managed "servers" which support protocols
- multi AZ 
- server per hour cost + data transferred cost 
- can use directory service or custom IDP 
- FTP -> internal VPC only 
- AS2 needs to be VPC internet / internal only 

FTP -> unencrypted file transfer
FTPS -> file transfer with TLS 
SFTP -> file transfer for SSH 
AS2 -> structured B2B data 

Identities -> directory service, custom ( Lambda / APIGW )
managed file transfer workflows -> serverless file workflow engine

Endpoint types: 
- Public: service is available to the public internet, only supports SFTP, there is a dynamic IP that can change, so use DNS, you can't control who has access
- VPC internet: uses SFTP, FTPS, AS2. accesible over DX, or VPC 
- VPC internal: uses STP, FTPS, SFTP, AS2. accessible over DX, or VPC

Questions 

Which storage types are resilient against AZ failure?
- EFS
- S3
- FSx for windows

Which storage types support POSIX?
- EFS
- EBS
- Instance store