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