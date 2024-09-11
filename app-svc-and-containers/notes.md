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