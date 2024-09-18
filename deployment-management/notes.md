# AWS Service Catalog

- Document or Database created by an IT team 
- organized collection of products 
- manage costs and allow service delivery to scale 

- portal for end users 
- launch predefined products 

# Elastic Beanstalk 

- PaaS 
- used for low infrastructure overhead
- fully customizable -> AWS products under the covers

Platforms 
- languages, docker & custom platform 
- go, java, tomcat
- .NET core ( linux ) & .NET ( Windows )
- Node, PHP, python and ruby
- single container docker, and multicontainer docker
- preconfigured docker
- custom platforms can be created when using Packer

Deployment policies
- how application versions are deployed in elastic beanstalk 

- all at once: brief outage, deploy to all at once 
- rolling: deploy in rolling batches 
- rolling with additional batch: capacity is kept the same during deployment
- immutable: all new instances with new version is created
- traffic splitting: fresh instances, with traffic split
- blue / green: you deploy two different sets of your application

# Elastic beanstalk and RDS

- can create an RDS instance within an EB environment 
- it's linked to the EB environment 
- if you delete the environment, you delete the RDS instance and you get data loss

- can create the RDS instance outside of the EB 
- add environment properties to point to RDS manually 

Steps that you must do if you want to decouple an RDS instance from an environment 
- create a snapshot 
- enable delete protection 
- create a new EB environment 
- ensure the new environment can connect to the DB 
- Swap environments ( CNAME or DNS )
- Terminate the old environment
- it will fail because you enabled delete protection, you need to manually delete the stack and retain the stuck resources

# advanced customization via EB extensions 

- way to customize EB environments
- inside the application source bundle, we create a .ebextensions, add YAML or JSON ending in .config, uses CFN format to create additional resources within the environment
- option_settings allows you to set options of resources 

# HTTPS with Elastic beanstalk 

- apply an SSL cert to the load balancer directly 
- or this can be added via the .ebextensions/securelistener.config

# Environment cloning with Elastic beanstalk 

- allows you to create a new environment by cloning an existing one 
- copies options, environment variables, resources, and other settings
- includes RDS but none of the data is copied


# OpsWorks 

- AWS managed implementations of chef and puppet

- 3 modes 
- Puppet enterprise - AWS managed puppet master server
- Chef automate - AWS managed chef servers 
- OpsWorks - AWS integrated chef, no servers 

key components: 
- Stacks: core component... container or resources which share a similar function 
- Layers: each layer is a specific function within a stack, this could be a DB layer or an app layer
- recipes and cookbooks -> these can be stored on github
- lifecycle events -> setup, configure, deploy, undeploy, and shutdown 
  - ex: the configure lifecycle event is typically run when any instances are added or removed from a layer in the stack 
- instances: compute instances managed by ops works, three types:
  - 24/7 -> always running 
  - time based -> used when you have a known period of time where you will have large load on system
  - load based -> turn on or off based on system metrics
 
# AWS systems manager

- managed and control AWS and on-premise infrastructure 
- agent based - installed on windows and linux AWS AMIs 
- manage inventory & patch assets 
- run commands & manage desired state
- parameter store -> configuration and secrets 

- instance role on an EC2 is required to use SSM 
  - can connect to systems manager endpoint via VPCE or an IGW

if using on-premise instances, you need: activation code, activation ID, and the IAM role for them to use 

# Systems manager -> run command

- allows you to take a command document, and run those documents on the instances 
- can be based on tags, resource groups, or instances 
- rate control -> concurrency and error threshold 
  - how many instances to run on at the same time 
  - how many instances can fail before the entire run command fails 

# Systems manager -> patch manager

Concepts
- patch baseline
- patch groups 
- maintenance windows 
- run command 
- concurrency & error threshold 
- compliance


predefined patch baselines -> 
  - AWS-[OS]DefaultPatchBaseline -> explicitly define patches 
  ex: AWS-AmazonLinux2DefaultPatchBaseline
  - Windows - AWS-DefaultPatchBaseline - critical and security updates
  - AWS-WindowsPredefinedPatchBaseline-OS - same as above 
  - AWS-WindowsPredefinedPatchBaseline-OS-applications - + MS app udpates 

patching architecture 
- define 1+ patch baselines 
- create patch groups ( these are defined with either tags or queries )
- create a maintenance window ( define schedule, duration, targets, and tasks )
- AWS-RunPatchBaseline runs with a baseline and targets ( this is the run command name )
- Inventory feature -> track that the patches were actually applied
