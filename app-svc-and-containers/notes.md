# EKS 

How to run EKS on aws?
- outposts
- EKS anywhere
- EKS distribution 
- in AWS on EKS

- control plane scales and runs on multiple availability zones 
- integrates with aws services.... ECR, ELB, IAM, VPC 

nodes:
- self managed
- managed node groups 
- pods on fargate -> you don't configure or scale or optimize anything, you create profile meaning pods can start on fargate

typical architecture: 

1. aws managed vpc: this is where the control plane runs from 
2. customer managed VPC: this is where your worker nodes are going to run 


# SNS 

- coordinates the sending and delivery of messages 
- messages are <= 256KB payloads
- SNS topics: base entity where permissions and configurations are 
- A publisher sends messages to a TOPIC, which has subscribers that receive messages 
- you can filter what messages subscribers receive from the topic

features:
- HA and scalable ( regional )
- delivery retries 
- delivery status 
- cross account deliver via TOPIC policy

# SQS

standard vs FIFO 
  - FIFO -> guarantees an ordering deliver of messages, exactly once deliver
    - 3000 messages per second with batching, up to 300 messages per second without
    - not as scalable 
  - Standards -> best effort so the messages can be received out of order, atleast once delivery

polling
- short polling -> immediate polling 
- long polling -> configure a wait time in seconds
  - you should be using long polling in SQS 

encryption
- at rest with KMS 
- encrypted in transit by default 

access:
- queue policy -> resource policy on the queue 
- IAM permissions 

billing
- billed on requests 
- 1 request can get 1-10 messages up to 64KB total
- the more frequently you poll, the less cost efficient it is 


- VisibilityTimeout -> amount of time that a client can take to process a message, if the client doesn't delete the message and the timeout is reached, it will be visible for other consumers on the queue 
- Dead letter queue -> where problematic messages can be sent for processing differently 
  - you can scale ASGs based on the amount of messages that are in a queue 

sqs extended client library 
- used when handling messages over SQS max 256KB 
- allow large payloads - stored in s3 
- Sends the message by uploading to s3, stores the link in the message

Delay queues
- allow you to postpone delivery of messages to consumers 
- configure a value called DelaySeconds, this means that messages added to the queue are invisible for the duration of time set by DelaySeconds
- maximum is 15 minutes 
- CANNOT USE WITH FIFO QUEUES 

dead letter queues 
- redrive policy: specifies the source queue, the dead letter queue, and the conditions where messages will be moved from one to the other, you need to define the maxReceiveCount
- allows you to configure an alarm for any messages that are delivered to the queue 

enqueue timestamp: this is the timestamp which will expire messages in a queue, this doesn't get reset when you move a message from a queue into a dead letter queue 
  - make the dead letter timestamp retention period longer than the normal queues 

# Amazon MQ 
- many orgs may already use topics and queues 
- open source message broker 
- based on managed apache ActiveMQ 
- JMS API...  AMQP, MQTT, OpenWire, and STOMP
- provides queues and topics 

- single instance or HA pair of instances 
- not a public service, Private networking is required 
- doesn't have native AWS integration 

- choose amazon MQ is you need to migration from an xisting system with little to no app changes 

# Lambda

Lambda networking 
- public ( default ) 
  - lambdas can access public AWS services, as well as the public internet 
  - this means that private VPC services cannot be reached by lambda functions
- VPC ( private )
  - when running in a VPC, they inject an ENI and they can access resources within a VPC
  - unless networking configuration allows external access, you can't access the internet 
  - vpc endpoints can allow you to connect to public AWS services 

Lambda permissions 
- execution role: the role that the code running in the lambda function, this will generate some temp permissions that the lambda function can use 
- resource policies: controls what services and accounts can invoke the lambda function

Logging 
- cloudwatch, cloudwatch logs and x-ray 
- logs from lambda executions go to cloudwatch logs 
- metrics -> invocations, retires, latency is stored in cloudwatch 
- x-ray -> for distributed tracing capabilities 

Invocation of functions
- synchronous invocation
- asynchronous invocation
- event source mappings

NOTE: event source mapping is a different type of trigger, this is where the lambda function is going to read from the configured source

synchronous 
  - CLI or API invokes a lambda function, and waits for the response 
  - this also happens if you're using lambda functions via API gateway
  - errors or retires need to be handled within the client

asynchronous 
 - usually when AWS services invoke lambda functions 
 - lambda is responsible for any reprocessing ( between 0 and 2 times )
 - lambda function code must be idempotent, you can run as many times as you want and the outcome will be the same

event source mappings
 - used with streams or queues which don't generate events
 - typically used on streams or queues which don't support event generation 
 - this is going to poll the lambda function 
 - the lambda function isn't being delivered an event, the lambda function is configured to read from that source

Lambda versions
 - the code + the configuration of the lambda function 
 - its immutable, and it never changes once its created 
 - $latest points at the latest version
 - Aliases point at that version - can be changes ( DEV, STAGE, PROD )

 - unqualified ARN: the lambda function without a version 


Lambda startup times
 - functions run in an execution environment which is a small container with resources allocated to it
 - cold start: full creation and configuration including downloading the function code
 - if you're running your function consecutively, it could use the same execution context, which will be known as a lambda warm start
 - provisioned concurrency: AWS will create and keep x contexts warm and ready to use which will improve your start speeds

Lambda function handler
- execution environment: the code and runtime that you're using to run your functions 

- INIT: creates or unfreezes the execution environment
- INVOKE: runs the function handler ( cold start )
- WARM START: the following invocations using the same environment which makes the execution quicker
- SHUTDOWN: terminates the environment, so the next invocation will be a cold start

Lambda layers
- you can split the function from the dependencies and share across multiple different functions
- allows libraries to be contained in a separate package 


# API gateway

request phase -> authorizes, validates and transforms 
response phase -> transform, prepare, and return to the client

Authentication
- Cognito user pools integration
  - passes the token into the request to API gateway
- lambda based 
  - client has a bearer token that is passed with the request 
  - lambda authorizer is called, does custom computations for authentication
  - IAM policy and identifier is passed back to API gateway
- IAM 
  - pass credentials into the header 

endpoint types 
 - edge-optimized - routed to the nearest cloudfront POP
 - regional - clients in the same region
 - private endpoint - accessible only from a VPC 

stages 
- apis are deployed to stages, each stage has one deployment 
- ex: dev, and production
- you can enable stages for canary deployment so a percentage of traffic is sent to the canary
- changes made in API gateway are not live, they must be deployed to a stage

- stages could be in the form of v1 / v2
- stage variables: this can be used to point at a different lambda function alias 
  - dev might point to $latest
  - after that we can point to versions 

deployments

errors
- 4xx - invalid request on the client side
- 5xx - server errors, valid request just a backend issue
- 400 - bad request, generic 
- 403 - access denied error 
- 429 - throttling is occuring on requests
- 502 - bad gateway exception 
- 504 - integration error 

caching
- caches by default for 300 seconds, and be max 3600 seconds
- calls only go to the backend when there is a cache miss
- cache can be encrypted 

Methods and resources 

resources 
- these are resources, they have different integrations 
- /listsomething -> you create methods in each of these resources ( GET, PUT, POST )
- /putsomething

- methods are where you configure your integrations

integrations 
- MOCK: used for testing, no backend environment
- HTTP: must configure api request and response 
- HTTP PROXY: pass through integration unmodified, return to the client unmodified 
- AWS: allows API to expose aws services, you configure request and response and setup valid mappings 
- AWS_PROXY: low admin overhead, you pass the request to the lambda function
- HTTP_PROXY: request passed straight through, no mapping template

OpenAPI and swagger 
- OpenAPI spec: used to be swagger, API description format for defining what APIs do
  - contains input and output parameters & authentication methods 
  - non tech information - contact info, licenes, terms of use, etc.

# AWS step functions 

- chaining lambda functions together gets difficult at scale
- perform a flow 
- standard workflow -> 1 year execution limit
- express workflow -> run for up to 5 minutes
- started via API gateway, IOT rules, eventbridge, lambda

- states: the things that are occuring inside a workflow 
  - SUCCEED & FAIL 
  - WAIT
  - CHOICE -> allows state machine to take a different path based on the input 
  - PARALLEL -> perform multiple sets of actions at the same time 
  - MAP -> accepts a list of things 
  - TASK -> represents a single unit of work performed by a state machine, can be integrated with many different AWS services 

# Simple workflow service 

- stepfunctions is the replacement for this service 
- build workflows to coordinate over distribution components 
- predecessor to step functions - uses instances and servers 
- 

# AWS mechanical turk
- managed human task outsourcing api - extend your app with humans 
- requesters to post human intelligence tasks 
- workers earn money by completing HITs 
- pay per task, perfect for tasks that are better suited for humans 
- qualifictations -> worker attribute, requires test and can be a requirement to complete HITs 

- data collection, manual processing, image classification 

# elastic transcoder and media convert
- file based video transcoding services 
- mediaconvert is soft of elastic transocder v2 
- serverless - pay for resources used 
- add jobs to pipelines ET or queues MC 
- file loaded from s3, processed, stored on s3 
- mediaconvert supports eventbridge for job signalling 

elastic transcoder is legacy, choose media convert

# AWS IOT
- IoT core is a suite of products, not one thing
- provisioning, updates, and control
- device shadow: virtual copy of a device that you can communicate with, a way to avoid unreliable connections  
- rules: event driven integration with other aws services 
  - match events on the messages that come from IoT devices

#  AWS greengrass
- extends some aws services to the edge
- compute, messaging, data management, and ML
- locally run lambda functions
- locally run containers
- IOT device shadows run locally 
- lambdas can access hardware locally