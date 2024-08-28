
Company needs hybrid DNS solution 
- all VPCs should be able to resolve cloud.example.com
- There is already an AWS direct connect connection between the on-premise corporate network and transit gateway 
- on premise systems should be able to resolve cloud.example.com

ANSWER: 
- Associate the private hosted zone with all VPCs.
- create a route53 inbound resolver 
- attach all VPCs to the transit gateway and create forwarding rules in the on-premise DNS server 

Company needs a solution that will give the ability to failover an API gateway with lambda functions, and a dynamodb table 

ANSWER:
- deploy a new api and lambda functions in another region 
- change the route53 record type to failover with target health monitoring 
- convert the dynamodb table to global tables


Difference between an allow list and deny list in AWS organizations?
- Allow list approach: only allows actions that are specified in the policy
- Deny list approach: allows everything, except for explicity denied actions 

