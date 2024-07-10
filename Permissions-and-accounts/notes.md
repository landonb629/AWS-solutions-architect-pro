# AWS organizations
- Management account in orgs:
  - invite existing accounts
  - accounts that join an org, they become member accounts
- OU (organizational units):
  - these are logical groupings of AWS accounts that can have certain policies applied to them

Consolidated billing:
  - member accounts in an organization will send their billing to the management account

Consolidation of reservations and volume discounts:
  - you can receive discounts by pooling together usage from all the member accounts in the organization
  - aws orgs treat all the accounts as one, so all accounts can receive the benefit of reserved instances

# Service control policies
Service control policies are not enabled in the management account by default, they must be enabled 


- they're applied in a descending manner 
- if you apply an SCP to the root account, they will be propogated to all accounts, if you have an OU called prod, and you apply to the prod OU, only the accounts or OU's under that will be impacted
- the management account is never impacted by SCP
- SCP never grants any permissions, it limits what IS and ISN'T allowed in an AWS account
- you're reducing what the account can do, so you're techincally restricting what the root user can do

Allow list vs deny list
- deny list: means that everything by default is allowed, and you can explicitly deny something
- allow list: means that everything by default is denied, and you have to explicity allow what you want to use

# STS - security token service 
`sts:AssumeRole` -> generates temporary AWS credentials
  - contains and access key and secret access key that expires
  - they're used to access AWS resources

Trust policy:
- this states who is allowed to assume a role
- can be an aws resource, or can be an identity 

Revoking IAM role temporary security credentials
- update the permissions policy on the assumed role to revoke sessions older than NOW 

# Policy interpretation
`NotAction` vs `Action`

Action: describes the specific actions that will be allowed or denied, ex: `Action: 'sqs:SendMessage'`

NotAction: explicity matches everything except the specified list of actions
  examples:
    - NotAction with Allow: provides access to all of the actions in an AWS service except for what is specified in the notAction

    this policy would allow everything for s3 besides deleted a bucket
    ```
      Effect: Allow
      NotAction: s3:DeleteBucket
      Resource: allS3buckets
    ```

    - NotAction with Deny: this policy would deny users access to all resources EXCEPT iam actions if they don't have MFA 
    ```
    Effect: Deny
    NotAction: iam:*
    Resource: *
    Conditon: ifNoMFA
    ```
    Effect Deny + NotAction can be thought of like two false boolean values being evaluated 
    if both == false then true 

# Permissions boundaries