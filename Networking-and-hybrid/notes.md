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

- global accelerator doesn't cache anything, it works at layer 4 (TCP / UDP)


# IPSEC fundamentals

- setup secure tunnels across insecure networks 
- between two peer networks (local + remote)
- provides authentication, and encryption while in transit

two phases: 
- phase 1 + phase 2 
IKE Phase 1:
- protocol for how keys are exchanged 
- authenticate using preshard key / certificate 
- IKE SA ( plase 1 tunnel )
IKE Phase 2:
- use the keys agreed in phase 1 
- agree on encryption method
- use key for bulk transfer 

# Site 2 Site VPN 
- connection between on-premise and VPC ( encrypted ) running over the public internet

- Can be fully HA if you design it correctly -> they're quick to provision

terms:
- customer gateway: the public interface on premise router, this gets created as a logical resource in AWS 
- Virtual private gateway: the interface on the amazon side that gets attached to a single VPC

Steps:
1. create a VPC 
2. Virtual private gateway
3. Customer gateway -> logical configuration that represents the public IP of the physical router on premise
4. VPN Connecion -> linked to VPG and CGW

static vs dynamic vpns
- dynamic: uses BGP, dynamic routing is discovered, communicates the state of links to the other peer, enabling route propogation on the route tables in the VPC
- status: static routes must be configured on the route tables 

VPN considerations:
- max throughput: 1.25GBPS
- all VPN connections: 1.25GBps 
- latency considerations: can be many hops between AWS and on-premise, so it's inconsistent 
- data transfer, and hourly cost 
- very quick to setup, much faster than setting up DX 
- can be used as backup for DX connection 

accelerate s2s vpn: utilizes the aws global accelerator network, the public internet is only used to get to the edge location, this is what provides lower latency and less jitter
  - virtual private gateways are not supported when using accelerated site to site VPN 

# Transit gateway refresher 
- connects VPCs to each other, and on-premise 
- network gateway, HA and scalable
- supports transitive routing 
- can be used to create global networks 
- can be shared between accounts with RAM 
attachments:
- VPC, S2S, direct connect gateway

TGW advanced concepts: 
- TGW has a main route table that holds all of the routes that are learned from the attachments
- can peer with up to 50 other transit gateways
- there is no route propogation over peering attachments ( tgw -> tgw )
- unique ASNs for future route propogation feature 
- public DNS -> private IP resolution doesn't work over peering attachments 

- association: route table is associated with an attachment
- propogation: configuration about which route tables are going to be populated 

if you need to isolate which VPCs can talk to which locations, you'll need to create another VPC, and make sure that the attachments aren't propogating their routes, but the destinations are propogating 

# Advanced VPC routing 
- route tables can be associated with an IGW or VGW

# AWS client VPN functionality
- managed implementation of OpenVPN 
- associated with one VPN 

client VPN endpoint: associate with one VPC, and multiple subnets inside VPCs
client VPN route table: contains the target networks, you can manually add routes as well

- you would most likely be creating a VPN VNET, and from there you can connect to the public internet, or peer with other VPCs
- client routing table will replace the local client route table

Split tunnel VPN: 
- allows clients to use their local route table, it just adds the routing to their local route table 

# AWS Direct connect concepts
- business premises -> DX location -> AWS region 
- DX = port allocation at a DX location, it's up to you to arrange for this to be connected
- port has an hourly cost & outbound data transfer
- provisioning time ( anywhere from 1-2 months ).... physical cables & no resilience 
- provides speeds from 1, 10, 100 GBps

customer premises router ---> DX location (this is a data center not owned by AWS) ---> AWS region ( public / private services )

## DX terms
- DX connection: physical port (1, 10, 100Gbps)
- you can't connect to DX with copper connections, fiber is a requirement
- autonegotiation is DISABLED, and port speed + duplex is manually set
- router must support BGP 
- optional use of macsec and bidirectional forwarding mode

## Connection process 
- create a DX request
- download LOA-CFA to initiate a cross connect between AWS and customer infra in the DX location
- cross connect is implemented by the data center staff 

## DX virtual interfaces 
- DX connections can have 50 public+private + 1 virtual interface on a direct connect connection
Virtual interfaces: allow you to run multiple l3 networks over layer 2 connection (direct connect)
  - this is what allows for VLANS and BGP peering sessions 

3 types of VIF:
- public: allow you to connect to public AWS services 
- private: used to connect to AWS private services + VPCs
- transit

# Direct connect -> private VIF 
- access 1 VPC resources using private IP addresses 
- attached to a virtual private gateway, must be in the same region where the DX location is created
- MTU of normal or jumbo frames are supported
- private VIFs that are terminating a virtual private gateway, you are able to use route propogation

Creating a Private VIF
- choose the virtual private gateway or direct connect gateway for terminating the connection
- you need to make sure that you are inputting the BGP ASN 
- choose your IP addresses, or have AWS generate that for you 
- you choose the VLAN, and this needs to match the customer side 

# Direct connect -> public VIF 
- used for accessing public zone services 
- you use this to access public AWS services
- no direct access to private VPC services
- you can advertise any public IP adresses you own over BDP
- you choose the VLAN, which needs to match the customer side
- you need to configure MD5 authentication 

you can be on premise, and access AWS public services, using the aws global network, not the public internet 

- public VIFs only allow you to access AWS IP ranges, this doesn't give you access to customer owned networks ( other businesses )

# public VIF + VPN 
- encrypted + authenticated tunnel 
- uses a public VIF + VGW/TGW public endpoints 
- this will allow you access to the VPC 

the VPN tunnel creates a private VPC connection between the customer gateway and an AWS VGW or TGW

# Direct connect gateway
- global network device that is accessible in all regions 
- create private VIF => associate the VGW attatched to VPCs globally
- VPC => on-premise communications allowed

- if you have a direct connect gateway that connects on premise to mutliple VPCs that are in different regions, this doesn't allow connection
between the two VPCs

- create a DX gateway which is a global AWS resource
- create a private VIF, terminate the private VIF into the direct connect gateway 

# Direct connect, transit VIFs, and TGW
- DX gateway can only do private VIF or transit VIF
- DX gateways don't route traffic through their connections, meaning if you have two TWG attachments, they cannot route between each other through the DXGW, they must create an attachment between them

- Transit gateway DOES allow routing across DX gateway attachments allowing connectivity between two hybrid locations

DX gateway does not alow routing between interfaces attached to the DX gateway

Transit VIF: should be used to access one or more AWS VPC transit gateways associated with direct connect gateways. you can use transit virtual interfaces with any direct connect dedicated or hosted connection

# DX resilience 
- DX locations are connected to the AWS region via redundant high speed connections

cross connect: cable between DX router and customer provided router in the direct connect location

How to add resilience?
- provision multiple DX connections, this will allow you to have resilience if one of the AWS DX routers goes down
- you can add multiple customer routers at the DX location
- you can add multiple customer routers at the on-premise location

- you can add two geographically different locations, with two different DX locations, and two different customer premises

most resilience: 
- two geographic locations
- two ports in each geographic location

# DX link aggregation groups
- allows you to take multiple physical connections and act as one
- max lag speed is 200GB

- this can help with cable resilience 
- this is a feature to help with adding speed

# Route 53 record types

NS -> allow delegation of zones to occur
A / AAAA -> map hostnames to IP addresses, A -> IPV4, AAAA -> IPV6
CNAME -> creates DNS shortcuts, these are host -> host mappings
MX -> used for sending email
TXT -> domain ownership 


# CNAME vs ALIAS
- you cannot CNAME apex domains

alias records fix this because when you get an ELB, you are referring to a domain name which would be a CNAME record

alias:
- maps a name to an AWS resource
- can be used for apex domain, or normal records


# route53 routing types
Simple routing -> there are no health checks

r53 health checks -> 
  - health checks are configured separately from the records in r53
  - default checks every 30 seconds
  - TCP, HTTP/HTTPS, also allows for string matching
  check types: 
    - endpoint check
    - cloudwatch alarm check
    - checks of checks ( calculated ), basically the status of other checks
  Notification:
    - can send to SNS when something is failed

Failover routing ->
  - use when you need to configure active / passive failover 
  - you create a primary and secondary record

Multi-value routing ->
  - meant to improve availability by allowing a client to resolve a DNS name to multiple different IP addresses
  - when you have many resources that can all handle requests that need health checks 

Weighted Routing ->
  - simple form of load balancing, when testing new versions of software
  - you add a weight in percent, which specifies how many times a record is returned

Latency Routing ->
  - use when trying to optimize performance 
  - the record which returns the lowest latency, is where users are routed
  - best for global applications

Geolocation Routing -> 
  - HAS NOTHING TO DO WITH CLOSEST RECORD
  - if you are in the UK and only a canadian record exists, you aren't going to be returned the record
  - you create records and tag them with the location of the country or state 
  - it returns applicable records, or the default, or no answer
    - applicable would be it checks if there are any records in the state, then country, then contintent, then default, then no answer
  - this type of record is useful for restricting content
  - if you create a US record, then only the people that are in the US can reach this location

Geoproximity Routing -> 
  - aims to provide records that are CLOSEST to your customer
  - this is different from latency routing, because this has nothing to do with connection speed, this has everything to do with the distance between you and the record
  - bias = allows users to increase or decrease the size of the geographic area, this will impact where resources are routed

# DNSSEC with route53 

DNS poisoning / spoofing -> hackers intercepting DNS queries and returning their own IP addresses so you are sent to the attacker machines 

Configuring DNSSEC for route53:
  - sign the records in your hosted zone with the private key in an asymmetric key pair
  - provide the public key from the key pair to the domain registrar

# Private link 
- provide or consume a service that is in another AWS account privately
- used to provide security access to service providers
two sides of relationship
- service provider -> VPC where the service is being provided
- consumers -> VPCs where you are consuming the service

- the service is injected into the VPC using interface endpoints, which are just ENIs that allows network connections to remain private

interface endpoint -> service provider VPC -> load balancer -> endpoint application services

example help:
- if you want HA, you will need to have multiple endpoints
- only supports IPv4 & TCP 
- private DNS is supported 
- DX, s2s, and VPC peering are supported

# VPC endpoints = Gateway + interface
PROJECT: create a VPC based serverless application that interacts with dynamodb through gateway endpoints
- provide private access to s3 and dynamodb 
- when you add gateway endpoints, prefix lists are added to the route table to look at the gateway endpoint
