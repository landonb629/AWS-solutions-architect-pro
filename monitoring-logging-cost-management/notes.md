# Cloudwatch 

- public service -> uses the public space endpoints 
- agent ingestion -> gives you richer metrics than just the service integration
- on-premise integration
- application integration

namespace -> container within cloudwatch for metrics, these are typically the name of the service 
  - ex: AWS/EC2, AWS/Lambda

datapoint -> timestamp, value, and optional unit of measurement ( % )

metric -> time ordered set of data points 
  - CPUUtilization, NetworkIn, DiskWriteByte

dimensions -> name / value pair when adding data points into cloudwatch 

resolution -> standard is 60s granularity... high is 1s ( they cost more )
 - as data ages, it's aggregated and stored for longer periods with less resolution

Alarm -> watches a metric over a time period 
  - ALARM or OK 
  - value of a metric vs threshold

data architecture 

namespaces -> containers for isolation 
  metrics -> ordered set of data points 
    dimensions -> allows you to look at specific metrics with name/value paires 

- certain AWS metrics allow aggregation across dimensions, for example CPUUtilization by instance type - NOT ALL and NOT CUSTOM

# Cloudwatch logs 

Ingestion
- getting logs into the system
- public service -> store, monitor, and access logging data
- AWS, on-premise, IOT, or any application
- CWAgent - system or custom application logging
- VPC Flow logs
- CloudTrail.. account events and AWS API calls

Log Group -> holds the log streams for a single resource, permissions, retention, and encryption is configured on the log group
  Log stream -> a sequence of log events, that share the same source
    Log event -> timestamp and raw message sent to a log stream

metric filters -> looks through the logs and can create a metric based on the filter
  - ex: looking for failed login attempts can create a metric based off a metric filter


Subscriptions -> used to get access ot real-time feed of log events from cloudwatch logs and have it delivered to other services such as kinesis stream, etc.
- real time delivery for a specific pattern, you set the destination and the permissions for cloudwatch to get access to the destination
- you can use kinesis data firehose to allow for realtime delivery of logging data to s3 

for real time log delivery 
- use an AWS managed lambda function to allow logging data to be delivered in realtime to Elasticsearch 
- use a custom lambda function to export data to nearly any destination in realtime
- kinesis data stream occurs in realtime allowing data to be used within other systems which integrate with KCL

Aggregation 
- we can use subscription filters to aggregate our data across accounts into a single account 
- we can send the data to a central kinesis data stream, and use firehose to persist it in kinesis data firehose


Summary -> 
- export logging to s3 and it won't be realtime 
- near real time is kinesis firehose 
- real time with lambda functions and kinesis data streams 

# Cloudtrail

- API calls or account activites are logged as cloudtrail events
- 90 days stored by default in event history
- create a trail for anything more than 90 days 
- regional service
- trails is how you store in CW and s3 

types:
- management events: control plane operations, ex: creating/deleting EC2 instance
  - managed by default
- data events: info about resource operations, ex: objects being uploaded to s3, lambda functions being invoked 

trail: unit of configuration for how cloudtrail should operate
  - can create all region, or single region trail 

iam, STS, cloudfront -> always log events to us-east-1
- global service events need to be enabled on a trail

organizational trail -> if you create from mgmt account of an organization, you can log all cloudtrail events in this trail, single point for all API events

# X-RAY

- distributed tracing application, designed for tracking sessions in an application
- gives you an overview of the flow in your application 
- tracing-ID is generated and inserted into a header
- segments: data blocks - host ip, request, response, work done, issues
- subsegments: more granular version of the above, calls to other services as part of a segment 
- service graph: JSON document detailing services and resources which make up your application
- service map: visual representation of the services used in your application

EC2 -> x-ray agent
ECS -> agent in tasks 
Lambda -> enable option
beanstalk -> per stage option 
SNS & SQS -> configured to send to x-ray 

- requires giving permissions in IAM 

# Cost allocation tags 
- enable for additional information for more billing information
- have to be enabled individually 
- AWS generated ( start with aws:createdBy )
- added to resources after they're enabled by AWS
- user defined and aws generated are both visible in cost reports and can be used as a filter

# AWS Trusted Advisor
draws upon best practices leared from serving AWS customers, inspects your AWS environment and makes recommendations when opportunities exist to save money, improve availability, security, and performance 
- Account Level: no agents to install, it just works 
- Cost optimization, performance, security, fault tolerance, service limits 
- 7 core checks with basic and developer support plans 

just remember these things: 
- free 7 core checks 
  - s3 bucket permissions - NOT objects 
  - security groups - specific ports unrestricted 
  - IAM  use 
  - MFA on root account
  - EBS public snapshots 
  - 50 service limits 

Business and enterprise support 
- 7 core checks, plus 115 further checks 
- cloudwatch integrations -> react to changes 