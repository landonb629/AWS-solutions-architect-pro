# Amazon workspaces
- virtual desktop as a service offering
- just like citrix or RDS just hosted in AWS 
- consistent desktop from anywhere - apps and state 

- uses directory service (Simple AD, AD connector) for authentication and user management (REQUIREMENT)
- Simple AD is best for POC 
- AD connector -> connect to on premise 

Networking:
- Workspaces don't run directly in the network, ENIs are injected into the customer managed VPC
- Workspaces use an ENI in a VPC... use VPC networking 
- hybrid connectivity is available, over VPN or Direct connect

Windows - can access fsx and EC2 windows resources

Not highly available by design -> can have single AZ failure 

Pricing:
- charged on a monthly basis or an hourly basis + the infrastructure cost

Infra required for creation:
- create the underlying VPC

# AWS Managed Microsoft AD 
- supports Group policy and SSO 
- supports schema extension - for AD aware apps (sharepoint, SQL server, DFS)
- HA by default -> one domain in two AZs atleast
- includes monitoring, recovery, replication, snapshots, and maintenance 
- supports one / two way trusts with on premise AD 
- supports RADIUS-based MFA 

You have a full directory service running in AWS that is independent of any other AD 

Sizes:
- standard: 30,000
- Enterprise: 500,000


When should you pick managed microsoft AD:
- need to support one way or two way trusts
- when you need to support RADIUS-based MFA 


# AD connector
- you must provide network connectivity to your on-premise instances (this would need to be setup with a direct connect or a VPN)
- Pair of directory endpoints running in AWS, like other directory services, there are ENIs injected into the customer managed VPC 
- no directory data is actually stored in AWS, meaning it relies on an existing on premise installation of MS AD
- resilient because it's deployed into two different VPCs

Sizes: impacts the throughput of the connectors
- small 
- large

- you can use multiple AD connectors to spread the load 


# AWS Control Tower 

- allow quick and easy setup of multi-account environments to implement aws best practices 

Very similar to AWS organizations, but there are many features that are added on: 
- IAM identity center
- Cloudformation
- Config
- Guard duty, security hub, security lake

- account factory: automating and standarizing new account creations, also allows for customization 
- guard rails: in the form of policies that dictate actions that aren't allowed, helps to standardize governance and compliance
  - these come in multiple forms: preventative, proactive, detective
    - preventative: implemented in the form of SCPs
    - detective: implemented in the form of aws config rules
    - proactive: implemented in the form of cloudformation hooks, that will stop code changes + other changes from happening before they are able to be committed to IaC