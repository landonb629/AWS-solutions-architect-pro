# DHCP in an AWS VPC 
- DHCP: dynamic host configuration protocol
  - how clients on a network are assigned an IP address from a pool of available IP addresses

DHCP Option sets: 
- cannot be edited once they are created, they're immutable
- can be associated with 0 or more VPCs
- VPCs can only be associated with 1 option set

default DNS service to use is the Route53 resolver that is to automatically the VPC subnet + 2 

ex: 10.100.0.2

if you would like to customize the domain name for a VPC, you will need to specify your own custom DNS servers to use within a DHCP option set

# VPC Router
- little direct control to the VPC router
- routes traffic between subnets 
- FROM EXTERNAL networks into the VPC and FROM THE VPC to external networks
- subnets are assigned to a single subnet at a time 


each subnet will receive it's own VPC router, this will be the first network address available in the subnet, and will be the default gateway for all network interfaces
- you can apply different route tables to each subnet, specifying different locations for routing 

Main route table: the default route table that gets assigned to each subnet

# Network access control lists

- NACL's are associated with subnets, not VPCs
- NACL's contain inbound rules and outbound rules
- rules are IP / IP range with ports
- offers explicit allow and explicit deny 

Rules are stateless.
- example: if you have a webserver that allows incoming requests on port 443, you're going to need to allow outbound requests on the ephemeral response port range

# Security Groups 
- SGs are stateful, they are going to detect response traffic and allow it, you don't need to create specific rules to allow traffic outbound

- there is no explicity deny, you can't choose to specifically create a deny rule, you can simply just NOT add a rule to allow 

- you cant block an IP or an IP range with a security group

- SGs are attached to ENIs, not the instances themselves

# AWS local zones 
- these give you the ability to place your applications in sub-locations within regions, for example: in us-west-2, you can place your resources in las vegas specifically in order to get your workloads closer to your clients

# BGP 101
- large network of autonomous systems -> these are routers controlled by one entity (viewed as a blackbox, an abstraction that BGP doesn't need to know about)

- ASNs are unique and allocated by IANA (0-65535), 64512-65532 are private

- operates over TCP port 179

- not automatic, peering is manually configured

- path-vector protocol: exchanges the best path to a destination between peers called paths 

iBGP: routing within an AS 

eBGP: routing between AS's 

# AWS global accelerator

- 2 anycast IP addresses
  - anycast: allow multiple devices to use a single IP address

- customers arrive at global accelerator edge location and are routed over the AWS global network backbone to their destination

- this service in turn brings the service closer to the customer and requires

- Global accelerator moves the actual aws network closer to the customer, not caching the assets like cloudformation 
