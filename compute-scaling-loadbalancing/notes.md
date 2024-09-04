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

# ELB architecture 

- AWS suggests that you use minimum /27 subnet, you need a minimum of 8 IP addresss

Cross zone load balancing -> allows any load balancer node to equally distribute traffic that it receives to any registered instances regardless
of what zone they're in
- this is now enabled as default 

# Session state 

- externally stored sessions: if you store your session state for your web app not local to the compute, you can free your load balancer to connect you to any registered instance without losing the state information

# ALB vs NLB 

ALB 
- ENCRYPTION TERMINATES AT NLB 
- Layer 7 load balancer -> listens on HTTP/S
- No other Layer 7 Protocols 
- No TCP/UDP/TLS listeners
- no unbroken SSL -> a new connection is created between ALB and your application 
- slower than network load balancers
- health checks can evaluate application health 
- if you need to forward encrypted connections through without termination, you need to use NLB 

NLB 
- TCP, TLS, UDP
- no visibility of HTTP or HTTPS
- No headers, cookies, session stickiness 
- 25% of ALB latency, they're super fast
- forward TCP to instances... so these have unbroken encryption ( they don't terminate on the network load balancer )

# Session stickiness 

- allows us to control which backend instances are used for a given user session
- 

when stickiness is enabled:
- the ALB generates a cookie called AWSALB, and this represents the user and their session, you define the duration 1 second - 7 days
- if the instance that the cookie maps to fails, a new instance will be used 
- if the cookie expires, it's removed and a new backend instance will be picked

Ideally, we would still want to externall store state

Problems:
- you can overload some of the servers if you have some users that are very heavy on use

# ASG - auto scaling groups

- auto-scaling and self healing for EC2 instances
- min size, max size, desired
- linked to a VPC, you set the subnets that are available

- ASGs are free, resources are not 
- use cooldowns to avoid rapid scaling 
- use more, smaller instances 
- 

launch configurations or launch templates: 
- launch config
- launch template

scaling policies: update the desired capacity based on specific metrics
- rules that adjust the vaules of an autoscaling group 
- manual scaling -> manually adjust the desired capacity 
- scheduled scaling -> time based adjustment, known periods of usage
- dynamic scaling -> 
  - simple -> "CPU above 50%"
  - step scaling -> more detailed rules, "add one if CPU above 50 but add 3 if CPU above 80"
  - target tracking -> "desired aggregate CPU 40%", not all metrics are accepted here, request count per target, CPU, network are allowed
- cooldown periods -> this allows your autoscaling group to stabalize and prevent it from launching or terminating additional instances before the effects of previous scaling activities are visible 

ASG + Load balancers
- using an ASG, instances will automatically be removed from a target group and added based on the scaling 
- ASG can use the load balancer health checks instead of the EC2 status checks. load balancer checks are application aware, EC2 health checks are not 

Scaling processes within an ASG
- launch and terminate -> SUSPEND and RESUME 
- AddToLoadBalancer -> addto LB on launch 
- AlarmNotification -> accept notification from CW 
- AZRebalance -> balances instances evently across all of the AZs 
- HealthCheck -> instance health checks are on/off
- ReplaceUnHealthy - terminate unhealthy and replace
- ScheduledActions 
- Standby - use this for instances when performing maintenance 

# ASG lifecycle hooks
- custom actions on instances during ASG actions 
- instance launch or instance terminate transitions
- eventbridge and SNS can be integrated for other eventdriven decisions 
if we define a lifecycle hook for launch:
- scale out activity happens
- instance moved into a pending state 
- instance moves into a pending:wait state
- we perform the custom actions on our instance
- moves to pending:proceed state
- moves to inService

if we define a lifecycle hook for terminate:
- scale in happens 
- instance moves to terminate:wait -> this will sit here for 3600 seconds or until the completeLifecycleAction runs
- moves to terminate:proceed
- moves to terminated

# ASG auto scaling policies 
- scaling policies are not required 

scaling based on SQS -> approximateNumberOfMessagedVisible

# ASG health checks

EC2 health checks 
- default 
- anything but RUNNING is seen as uhealthy

ELB health checks
- healthy is RUNNING and passing the ELB health checks
- this can be application aware, because it can check a custom route


Custom
- instance can be marked healthy and unhealthy by an external system

Health check grace period -> delay before starting the checks, default is 5 minutes


# Connection draining & deregistration delay 

connection draining 
- what happens when instances are unhealthy or deregistered 
- the normal behavior would be all connections are closed & no new connections are allowed. with connection draining, in-flight requests are allowed to finish
- CLASSIC LOAD BALANCERS ONLY

deregistration delay
- supported on ALB, NLB, and GWLBs 
- defined on the target group - NOT the LB 
- stops sending requests to deregistering targets but allows the existing connections to continue until they complete naturally, or the deregistration delay gets reached  

# X-forwarded-for & PROXY protocol 

Problem: when we add a load balancer to the architecture, we lose the ability to see the source IP address of the client

x-forwarded
- http header ( layer 7 )
- added or appended by proxies or LBs, as the left most header in the list
- any other proxies that need to be passed through will have the header appended
- not supported on NLB 

PROXY protocol
- works on layer 4, this is a layer 4 header ( works with HTTP and HTTPS )
- v1 is CLB and NLB is v2 which is binary encoded

# EC2 placement groups 

Cluster - packs instances close together inside an AZ. this enables workloads to achieve the low latency network performance necessary for tightly-coupled node-to-node communication that is typical of HPC applications 
  - same rack, and sometimes the same EC2 host
  - 10 GBPS stream possible, 5GBPS normally 
  - can't span AZs, only possible with a single AZ 

Partition - spreads your instances across logical partitions such that groups of instances in one partition do not share the underlying hardware with groups of instances in different partitions. this is used in largely distributed and replicated workloads, such as hadoop, cassandra, and kafka 
  - can be created across multiple AZs in a region 
  - difference between spread and partition, is that partition can shared EC2 instances on the same hardware

Spread - strictly places a small group of instances across distinct underlying hardware to reduce correlated failures
  - maximum amount of availability and resiliency 
  - each instance is placed on its own rack with it's own power
  - you only can have 7 instances per AZ in a single region 
  - provides a lot of infrastructure isolation 
  - cannot use dedicated instances or hosts

# Gateway Load Balancers

help you run and scale 3rd party appliances. things like firewalls, intrusion detection and prevention systems 
- inbound and outbound traffic ( transparent inspection and protection )
- gateway load balancer endpoints: traffic enters and leaves these endpoints
- GWLB balances across multiple backend appliances
- traffic is tunneled using GENEVE protocol 
Flow stickiness: one flow will always use the same appliance, this allows monitoring the flow 

typical architecture:
- uses security VPC and application VPC 

1. client accesses web application which arrives at the internet gateway, which has an ingress route table 
2. internet gateway will translate the destination from public to private, traffic will be sent to gateway load balancer endpoint 
3. packets are forwarded to gateway load balancer in security VPC, they are sent to the securty appliances
4. packets are returned to the GWLB and sent back to the GWLBE 
5. packets are sent to the ALB and then the application

# difference between launch template and launch configuration
