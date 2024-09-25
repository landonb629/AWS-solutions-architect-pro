# Amazon guardduty

- continuous security monitoring service 
- analyzes support data sources, uses AI/ML, plus threat intelligence feeds 
- identifies unexpected and unauthorised activity

- notify or event-driven protection / remediation 

# AWS Config

- record changes over time on resources 
- audit of changes, and if your resources are compliant with standards 
- DOESNT PREVENT ANYTHING FROM HAPPENING

Regional service -> has cross region support and account aggregation 
- changes can generate SNS notifications and near realtime events via eventbridge & Lambda

# AWS inspector 

- Scans EC2 and the OS
- scans containers 
- finds vulnerabilities 
- provides a report of findings ordered by priority 
- network assessment can be agentless 
- host assessment ( requires an agent )

- network reachability -> no agent required, but an agent is available which can add additional OS visibility 


# KMS 
- region and public service 
- create, store and manage cryptographic keys
- KMS is FIPS 140-2 level 2 compliant
- KMS key -> ID, date, policy, description and state ( logical ), they are backed by physical key material 

- Data encryption keys -> different type of key derived from a KMS key, they're generated using the GenerateDataKey, use these when you're encrypting data larger than 4KB 

symmetric -> uses the same key for encryption and decryption
asymmetric -> uses two different keys , a public key and a private key for decryption

Key concepts: 
- KMS keys are isolated to a region and they never leave 
- keys are either AWS owned or customer owned keys
- aliases are shortcuts to keys 

key policies and security: 
- key policies ( resource policy ) -> all kms keys have one of these, keys must be told that they trust the account that they're contained in 
- key policies + IAM policies -> we can use a combination of these to allow users to manage keys but not allow them to perform cryptographic operations with these keys 

# CloudHSM

- single tenant hardware security module 
- AWS provisioned -> fully customer managed
- AWS CANNOT access the HSM
- FIPS 140-2 level 3 -> IF YOU NEED LEVEL 3 THEN YOU MUST USE CLOUDHSM

- you access with industry standard APIs -> PKCS#11, JCE, CryptoNG

- KMS can use cloudHSM with custom key store, Cloud HSM integration with KMS

- CloudHSM is deployed into it's own CloudHSM VPC
- Not HA by default, you need to configure a cluster to get HA 
- in order to interface with cloudHSM, they're injected into your VPC with Elatic network interfaces
- in order to interface with them, you need to use the cloudHSM client which must be installed

use cases for cloudHSM: 
- no native integration between cloudHSM and other AWS services
- offload SSL/TLS processing for web servers 
- Enable transparent data encryption for oracle databases 
- protect the private keys for an issuing certificate authority 

# AWS secrets manager

- designed for secrets, passwords + API keys 
- usable from the console, API, and SDKs
- supports automatic rotation of secrets
- directly integrates with some products like RDS

# VPC flow logs 
- only capture packet metadata 
- if you need to capture the content of the packets, you're going to need a sniffer

virtual monitors are applied at three levels:
- VPC: all enis in the vpc 
- subnet - all enis in  that subnet 
- ENIs directly 

- flow logs are not real time 
- you can send the logs to cloudwatch or s3
- you can query from athena for querying 

# Application layer firewalls ( layer 7 )

Normal firewalls run on layer 3 4 and 5 

layer 7 firewalls 
- they can see all the lower layers including HTTP
- it can see headers, data, hosts, etc.
- can identify normal or abnormal requests that are protocol specific 

# WAF 
- implementation of a layer 7 firewall 
- you have a web ACL that has rule groups 

protects the following resources: 
- cloudfront 
- ALB 
- app sync 
- api gateway

- outputs logs that can be sent to s3 ( every 5 minutes )
- you can use firehose for realtime response to send to s3 

Web access control lists 
- default action ( allow or block ) - non matching 
- resource type -> cloudfront or regional service ( alb, app sync, etc. )
- add rule groups or rules which are processed in order 

Web ACL capacity units -> indication of the complexity of rules - default max 1500 
- web acls are associated with resources 

Rules:
- type, statement, action -> this is the high level outline of what a rule looks like 
- type: regular, or rate based
  - rate based would be something like 5000 ssh connections within 5 minutes 
- statement: what the rule should match, or what the count needs to be
  - rules can have AND OR NOT logic 
  - can look for matches on origin IP, label, header, cookies, query parameters, URI path, query string, body, HTTP method
- action: 
  - allow, block, count, require captcha, or a custom response, labels can be added for multistage flows 

Pricing:
- monthly price for each web ACL -> 5/month
- Rule on WEBACL - 1/month 
- requests per WEBACL - $.6 every 1 million requests 
- intelligent threat mitigation
  - bot control - 10/month, and every million requests
  - captcha - /1k challenge attempts 
  - fraud control / account take over - /month price, and /1000 login attempts 

# AWS Shield 

- DDOS mitigation
- standard + advanced 
- attack types:
  - network volumetric attacks ( layer 3 )
  - network protocol attacks (layer 4 )
  - web request floods ( layer 7 )

standard ->
- free for AWS customers 
- common network ( layer 3 ) or transport ( layer 4 )

advanced -> 
- has a cost associated 
- $3000 per month per org for a year + the data out 
- protects cloudfront, route53, accelerator, ALBs, and any EIPs 
- not automatic -> these must be explicitly enabled on the resources 
- cost protection -> if you incur costs against attacks that get through, then AWS will cover the cost 
- proactive engagement and the AWS shield response team 
- WAF integration -> includes basic WAF fees for web ACLS, rules and web requests 
- real time visibility of DDOS events and attacks 
- health based detection with route53 health checks 