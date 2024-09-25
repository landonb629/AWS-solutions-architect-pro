# Types of DR 

- backup and restore
  - longest time to get back up and running 
  - you have only the backup media, but no ongoing spare infrastructure 

- pilot light 
  - you run a second site which is preconfigured 
  - only has bare minimum of things running, might be running a copy of a database, or some components that are shutdown like servers
  - used when we need to have a plan but it's very unlikely that these disasters occur 

- warm standby
  - minimized version of your infrastructure running in another environment 
  - ex: instead of running XL instances, you're running smalls 

- active / active 
  - complete replica of your environment running 
  - this environment is running at all times 

# DR storage

EBS:
- EBS replicates within an AZ 
- failure of an AZ means the failure of a volume though
- you can store snapshots in s3 which would make them AZ reliable
S3:
- replicated across multiple AZs 
- provides regional resilience 

EFS:
- a file system is replicated across AZs 
- regional resilience 

# DR compute 

EC2:
- runs in one AZ on one host 
  - host failure can impact an EC2 instance
  - as well as AZ failure 
- auto scaling groups are what helps us with the failures 

# DR databases 

DynamoDB: 
- data is replicated in to multiple AZs 
- this is regionally resilient 
- global tables gives you the ability to be resilient throughout different regions, this is multi master 

RDS: 
- you define a subnet group which defines what subnets to run RDS in 
- primary instance will replicate to stand by and failover in case of failures
- provides async cross region replication
Aurora: 
- not limited to primary and standby
- you can have more than one replica in each AZ 
- cluster storage architecture -> the instances are sharing throughout the cluster, spread across every AZ 
- aurora global, can have read write in one region, and just read in another region

# DR Networking

- subnets are tied to AZs, so if the AZ fails then the subnet fails 
- load balancers are regional services, so this will help us provide the HA 
- interface endpoints are tied to a subnet, so you need to provision multiple interface endpoints to get HA

Global Networking
- R53  is a global service that can survive multiple region failures 
- you can use R53 with regional endpoints with health checks that can remove routing for failed regions