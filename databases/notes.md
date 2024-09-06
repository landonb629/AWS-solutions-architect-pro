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

How is instance mode different than cluster mode? 