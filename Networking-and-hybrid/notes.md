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