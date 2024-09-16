# The 6 R's of Cloud migration

- discover, assess and prioritize 
- rehosting - lift and shift
- replatforming - lift and shift with some optimization
- repurchasing - move to something new 
- refactoring / rearchitecting - take advantage of cloud 
- retire - do you even need it 
- retain - not worth the money or time to migration but keeping 

rehosting 
- migrate the application as is
- reduce admin overhead 
- potentially easier to optimize when in AWS 
- cost savings 

use vm import / export and server migration service 


replatforming 
- rehosting but with optimization
- you might use RDS instead of self managed DBs 
- you might use ALB instead of self hosted load balancers 
- you might use s3 instead of media storage or backup 


repurchasing 
- unless you have to self manage an application 
- use a PaaS or anything other aaS

refactoring / rearchitecting 
- reviewing the architecture of an application
- adopting cloud native architectures and products 
- service oriented or micorservices 
- APIs, event driven, or serverless 


# Virtual machine migrations in AWS 

application discovery service 
- discover on-premise infrastructure 
- two modes:
  - agentless mode: uses ova virtual appliance that integrates with VMware, measaures performance and vm usage
  - agent-based mode: discovery for in VM data gathering, network, processes, usage, and performance
- builds dependencies between servers 
- integration with AWS migration hub and athena 

server migration service
- migrate whole VMs into AWS 
- agentless, uses a connector which is VM that runs on-premise 
- integrates with VMware, hyper-V and AzureVM only
- incremental replication of live volumes 
- orchestrate multi-server migrations 
- creates AMI which can be used to create EC2 instances 

# Database migration service ( DMS )

- runs a replication instance which runs in EC2
- source and destination endpoints point at 
  - source and target databases 
- one endpoint must be in AWS 

task: moves data from source to target using source endpoint and destination endpoint 
- full load: one off migration of all data 
- full load + CDC: migrates existing data and captures changes that happen after the full load migration
- CDC only: replicate only data changes 

schema converstion tool ( SCT ): 
- performs schema converstions between different database engines 
- used for large migrations, including DB -> s3 
- not used when migrating engines of the same type 

DMS and snowball 
- larger migrations might be multi-TB in size 
- DMS can utilise snowball

1. use the SCT to extract data and store locally and move to a snowball device
2. ship the device back to AWS. they load into an s3 bucket
3. DMS migrates from s3 into the target store 
4. CDC can capture changes, via s3 intermediary they are also written to the target database 

# Storage Gateway - Volume stored / cached 

- runs as a virtual machine on premise ( can run as a hardware appliance )
- presents storage using ISCSI, NFS, or SMB
- integrates with EBS, S3, and Glacier within AWS 
- migrations, extensions, storage tiering, DR, and replacement of backup systems

stored volume mode:
- everything is stored locally
- the storage gateway presents storage over the network similar to a SAN or a NAS
- upload buffer: any data that is written to the local storage is also writter to this buffer, which is then async copied up into aws, and written to EBS volumes
- this is over the storage gateway endpoint which is a public endpoint 

use case: 
- full disk backup for servers 
- doesn't allow extending your data center capacity 
- use when you need low latency access to data 

cached mode:
- primary location of data is in s3 instead of running locally on the gateway, it's in an aws managed portion of s3
- the data is cached locally
- this allows for data center extension
- slower to access but the capacity can be extended to AWS

# Storage Gateway - virtual tape library mode 

- storage gateway in VTL mode presents itself to the backup server like a tape drive with a media changer
- it has a local cache and an upload buffer, instead of using physical tapes
- VTL = s3, virtual tape shelf = glacier 
- virtual tapes can be 100GB -> 5TB 
- the storage gateway can use 1PB across 1500 virtual tapes 
- virtual tape shelf has unlimited storage due to the fact that glacier is going to be archived

# Storage gateway - File mode

- managed files 
- bridges on premise file storage to s3
- create mount points available via NFS or SMB
- does read and write caching to ensure high performance 
- you can integrate with active directory authentication

notifyWhenUploaded: sends an event to the other storage gateways when there has been a new file uploaded


# The AWS snow services 

designed for moving large amounts of data IN and OUT of AWS 
- 50TB to 80TB 
- 10TB to 10PB economical range for moving data into AWS 


snowball -> does not have any compute availability 

snowball edge -> comes with storage and compute
  - larger capacity vs snowball 
  - you get faster than snowball 
  - storage optimized with EC2, storage optimized, and compute with GPU

snowmobile
- portal DC within a shipping container on a truck 
- specially ordered from AWS 
- ideal for single location with 10PB 
- up to 100PB per snowmobile

# AWS data sync
- data transfer service TO and FROM aws 
- migrations, data processing, archival, or DR/BC
- designed to work at huge scale
- keeps metadata ( permissions and timestamps )
- built in data validation

- datasync agent runs in a virtualized platform and communicates with the Data sync endpoint
- communicates over NFS or SMB
key features: 
- 10GBps per agent 
- bandwidth limiter ( avoid link saturation )
- integration with AWS services 

components:
- task -> job within data sync and defines the to and from 
- agent -> software that is used to read or write to onpremise data stores 
- location -> to and from location for every single job 