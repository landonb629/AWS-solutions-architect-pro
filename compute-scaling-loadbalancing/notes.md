# Regional and Global Architecture
global
- global service location and discovery
- content discovery and optimisation 
- global health checks and failover 

regional
- regional entry point
- regional scaling and resilience 
- application services and components

# EC2 purchase options ( launch types )

- on demand -> 
  - multiple customers will be able to run isolated EC2 instsances on shared hardware 
  - per-second billing while the instance is running 
  - no upfront cost 
  - no discount 
  - default purchase option

- spot ->
  -  pricing is based on spare capacity at a given time
  - if spot price goes above your max price, your instances are terminated
  - only use for workloads that can tolerate interruptions
  - anything which is stateless 

- Reserved ->
  - long term consistent usage of EC2 instances
  - unused reservations are still billed 
  - reservations are locked to an availability zone or region
    - AZ -> reserved capacity
    - region -> can be used in any AZ 
  - partial coverage of larger instance is supported 
  - can commit for 1 year or 3 years
  - payment structure
    - no upfront -> reduced per/s fee, least discount amount 
    - All upfront -> no per second fee 
    - partial upfront -> reduced per/s fee
  
  - Scheduled reserved instances -> when you need long term usage without running constantly
    - minimum period is 1 year for purchase 
    - doesn't support all instance types or regions

  - Capacity reservations -> when you have compute that can't tolerate interruptions
    - regional reservations provide a billing discount for instances in any AZ in that region, but they don't reserve any capacity within that AZ
    - zonal reservations -> only apply to one AZ but they also get you the capacity reservations 
    - On demand capacity reservations -> booked to ensure you always have access to capacity in an AZ but at full on-demand pricing, no term limits but you pay whether or not you end up consuming the capacity
  
  - EC2 savings plan 
    - Hourly commitment for 1 to 3 year term 
    - a reservation of general compute amounts would be able to be used for any compute in AWS ( ec2, fargate, lambda )
    - EC2 savings plan -> this gives you flexibility on size & OS but can only be used on EC2 instances 
  
- dedicated host
  - EC2 host that is dedicated to you in it's entirety 
  - no instance charge because you're paying for the host itself
  - you need to manage the capacity on your hosts
  - do this if you have licensing based on sockets/cores
  - host affinity -> links an instance to a host, if it's restarted it will remain on the host

- dedicated instances
  - you don't pay for the host, but you don't share the host
  - you pay a one off hourly fee for any region you're using these
  - common in industry standards where you can't share infrastructure
  - when you can't share hardware but you don't want to manage the entire host

### Summary of EC2 purchase options
- on-demand -> regular option for running workloads
- spot -> run on unused host capacity, set a max price, if price goes above, your instances will be terminated 
- reserved -> AZ or Region reservation, in AZ reservation you get a capacity reservation as well, not with regional.
- capacity reservations -> reserves capacity in an AZ but you pay the on-demand pricing for thecompute, and you will pay for the capacity reservation as well
- compute + EC2 savings plan -> compute savings plan can be used on multiple resources, EC2 savings plan is just for EC2 instances
- dedicated host -> an entire host at AWS, you need to manage the capacity, do this when you have socket / core licensing requirements 
- dedicated instances -> 


# Advanced EC2 Networking 

- instances are deployed with a primary ENI which cannot be removed
- additional ENIs can be added in and removed from other subnets but not in any other AZs
- the ENIs are where the security groups are attached, not the actual instance itself

- larger instances typically allow for more secondary ip addresses 
- public IP address isn't visible to the ENI, ip traffic gets translated to and from the VPC by the internet gateway 
- in order to get a static IP address, you need to provision an elastic IPv4 address

- if you enable IPv6 addressing, those are all publicly routable. 

- management or isolated networks -> because we can attach multiple network interfaces to an EC2 instance, we can have a public network interface but we can also have a private network interface that is only available for internal staff

- you can use different network interfaces for security or networking appliances 

# Bootstrapping and AMI baking

Ready for service lag -> the time when autoscaling is enacted, and the EC2 services actually become ready to serve traffic 

- Bootstrapping 
  - provision an EC2 instance, add a script to the user data, and the script will run when the instance is launched

- AMI baking 
  - launch a master EC2 instance, perform all the time consume tasks upfront, create an AMI from the configured instance
  - this is much faster to get the EC2 ready to go 
  - harder to adjust things once the AMI is baked 

How to use both of them? 
- provision a base EC2 instance
- perform the configuration of the baseline 
- create an AMI 
- customize the base AMI with user data