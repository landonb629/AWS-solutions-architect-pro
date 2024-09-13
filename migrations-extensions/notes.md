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