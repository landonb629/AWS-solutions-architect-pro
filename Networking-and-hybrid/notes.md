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


