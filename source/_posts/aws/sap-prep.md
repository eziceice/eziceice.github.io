---
title: AWS Solution Architect Professional
date: 2021-08-17 12:00:52
tags: aws
categories:
- cloud
---
## Identity & Federation

### IAM

- Explicit DENY overrides ALLOW
- When you assume a role, you give up your original permissions and take the permissions assigned to the role.

### STS

- AssumeRole with **ExternalID**

### Identity Federation

- Amazon SSO is the new managed and simpler way to replace SAML 2.0 Federation.
- Use Custom Identify Broker Application only if identity provider is not compatible with with SAML 2.0.
- Web Identity Federation - Not recommended by AWS - use Cognito instead.

### AWS Directory Services

- AWS Managed Microsoft AD (Users are created in on-prem AD and AWS managed AD)
- AD Connector (Proxy - all users are still managed in on-prem AD)
- Simple AD (All users managed in AWS)

## Security

### CloudTrail

- CloudWatch Events
- CloudTrail Delivery in CloudWatch Logs
- CloudTrail Delivery in S3

### SSL Encryption, SNI & MITM

- SNI solves the problem of loading multiple SSL certificates onto one web server
- Only works for ALB & NLB and CloudFront
- Route53 supports DNSSEC for DNS service

### ACM

- ACM is a regional service

### CloudHSM

- AWS provisions encryption hardware, dedicated hardware
- You manage your own encryption keys entirely, not AWS

## Compute & Load Balancing

### AWS Lambda

- $Latest version is mutable
- Published version is immutable and all versions can be invoked differently
- Alias is mutable and can be used as Canary deployment

### Elastic Load Balancer

- NLB has one static IP per AZ, and supports assigning Elastic IP
- NLB works with VPC endpoint/PrivateLink
- ALB can have Lambda as target group, NLB can't
- ALB cross-zone load balancing is always enabled with no additional charges
- NLB cross-zone load balancing is disabled by default
- Stickiness doesn't work for NLB

### API Gateway

- API gateway has a 10MB payload size limit
- API gateway has a 28 seconds timeout maximum

### Route 53

- A: hostname to IPv4
- AAA: hostname to IPv6
- CNAME: hostname to hostname
- Alias: hostname to AWS resource
- Route 53 health checks can monitor CW alarms and integrate with CW metrics

## Storage

### EBS & Local Instance Store

- gp2: 3 IOPS/GiB, minimum 100 IOPS, burst to 3000 IOPS, max 16000 IOPS
- io1: Min 100 IOPS, Max 64000 IOPS (Nitro) or 32000
- st1: Throughput Optimised HDD, 500 GiB - 16 TiB, 500 MiB/s throughput
- sc1: Cold HDD, Infrequently accessed data, 250 GiB - 16 TiB, 250 MiB/s throughput
- RAID Configurations: 0, write to either volume. 1, write to both volume.

### S3
- At least 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD requests per second per prefix in a bucket

## Caching

### CloudFront

- Signed URL = access to individual files
- Signed Cookies = access to multiple files
- Cache based on:
    - Headers
    - Session Cookies
    - Query String Parameters
- Cache lives at each CloudFront Edge Location

### Redis vs Memcached

- Redis:
    - Multi AZ with Auto-Failover
    - Read Replicas to scale reads and have high availability
    - Persistent, Data Durability, Read Only File Feature, backup and restore features
- Memcached:
    - Multi-node for partitioning of data (sharding)
    - Non persistent
    - No backup and restore
    - Multi-threaded architecture

## Databases

### DynamoDB

- Maximum size of an item is 400KB
- LSI - Local Secondary Index
    - Keep the same primary key
    - Select an alternative sort key
    - Must be defined at table creation time
- GSI - Global Secondary Index
    - Change the primary key and optional sort key
    - Can be defined after the table is created

### Aurora

- Global Aurora Database - recommended
    - 1 Primary Region
    - Up to 5 secondary regions, replication lag is less than 1 second
    - Up to 16 read replicas per secondary region
    - Helps of decreasing latency
    - Promoting another region (for disaster recovery) has an RTO of < 1 minute\
- Aurora Multi-Master
    - Immediate failover
    - Every node dose RW
- [RDS Multi-Region Failover](https://raw.githubusercontent.com/eziceice/blog/master/aws/RDS-multi-region-failover.png)

## Service Communication

### Step Functions - Tasks

- Lambda Tasks
    - Lambda function
- Activity Tasks
    - Activity worker (HTTP), EC2 Instances, mobile device, on premise DC
    - They poll the Step functions service
- Service Tasks:
    - Connected to a supported AWS service
    - Lambda function, ECS task, Fargate, DynamoDB, Batch job, SNS & SQS
- Wait Task:
    - To wait for a duration or util a timestamp

### SWF - Simple Workflow Service

- Step functions is recommended to be used for new applications, except:
    - if external signals to intervene in the processes is needed
    - if child processes that return values to parent processes is needed
    - If Amazon Mechanical Turk is needed

### SQS

- Message size of max 256KB (use metadata in S3 for large messages)
- Messages can be processed twice by consumer (in case of failure, timeouts, etc). Idempotency at the consumer level is very important.
- Set the queue visibility timeout to 6 times the timeout of your lambda function

## Data Engineering

### Kinesis Data Streams

- Kinesis Streams: low latency streaming ingest at scale
- Multiple applications can consume the same stream (pub-sub model)
- Real-time processing with scale of throughput
- Once data is inserted in Kinesis, it can't be deleted, will stay there until the data retention period is gone.
- Producer:
    - 1mb/s or 1000 messages/s at write PER SHARD, otherwise "ProvisionedThroughputException" (add shards)
- Consumer Classic:
    - 2mb/s at read PER SHARD across all consumers
    - 5 API calls per second PER SHARD across all consumers
- Consumer Enhanced Fan-Out:
    - 2mb/s at read PER SHARD, PER ENHANCED CONSUMER
    - No API calls needed (push model)
- Data Retention:
    - 24 hours by default and can be extended to 7 days

### Kinesis Analytics

- Kinesis Analytics: perform real-time analytics on streams using SQL

### Kinesis Firehose

- Serverless, auto-scaling service - **near real time (Because of the firehose buffer)**
- Kinesis Firehose: load streams into S3, Redshift, ElasticSearch & Splunk
- Destinations:
    - AWS destinations:
        - S3
        - Redshift (Copy through S3)
        - Elastic Search
    - 3rd Party Destinations:
        - Splunk
        - New Relic
        - etc.
    - Custom Destinations:
        - HTTP endpoint
- Source records && failed records can be sent to S3 as well

### Redshift

- Leader node: for query planning, results aggregation
- Compute node: for performing the queries, send results to leader

## Monitoring

### CloudWatch

- CloudWatch Logs Agent
    - Old version of the agent
    - Can only send to CloudWatch Logs
- CloudWatch Unified Agent
    - Collect additional system-level metrics such as RAM, processes, etc...
    - Collect logs to send to CW logs
    - Centralised configurations using SSM Parameter Store
- Both agents cannot send logs to Kinesis (have to use KCL)

## Deployment

### CloudFormation

- DeletionPolicy
    - Retain: Specify on resources to preserve/backup in case of CFN deletes
    - Snapshot: Create a Snapshot before CFN deletes
    - Delete(Default)
        - RDS:DBCluster resources default policy is Snapshot
        - Delete an s3 bucket you need to first empty the bucket of its content
- CloudFormer: generate CFN from existing resources

### OpsWorks

- For chef/puppet stacks only
- Can manage ELB and EC2 instances
- Cannot manage an ASG

### System Manager

- SSM Patch Manager:
    - Define a patch baseline
    - Define patch group
    - Define Maintenance Windows
    - Add the AWS-RunPatchBaseline Run Command - works for Linux and Windows
    - Define Rate Control
    - Monitor Patch Compliance using SSM Inventory

## Cost Control

### Trusted Advisor

- Can check if an S3 bucket is made to public
    - but cannot check for S3 objects that are public inside of your bucket
    - Use CloudWatch Events/S3 Events instead
- Service Limits
    - Limits can only be monitored in Trusted Advisor (cannot be changed)
    - Cases have to be created manually in AWS Support Centre to increase limits

### EC2 Instance Launch Types

- On Demand
- Spot Instances
- Reserved (MINIMUM 1 year)
- Dedicated Instances: no other customers will share your hardware
- Dedicated Hosts: book an entire physical server, control instance placement
    - Great for software licenses that operate at the core, or socket level
    - Can define host affinity so that instance reboots are kept on the same host

## Migration

### Storage Gateway

- Bridge between on-premise data and cloud data in S3
- File Gateway
    - appliance is a virtual machine to bridge between your NFS and S3
    - configured S3 buckets are accessible using the NFS and SMB protocol
    - Most recently used data is cached in the file gateway
    - Can be mounted on many servers
- Volume Gateway
    - Block storage using iSCSI protocol backed by S3
    - Cached volumes: low latency access to most recent data, full data on S3
    - Stored volumes: entire dataset in on premise, scheduled backups to S3
    - Can create EBS snapshots from the volumes and restore as EBS
- Tape Gateway
    - You can't access single file within tapes, the entire tape need to be restored

## VPC

### VPC Peering

- Overlapping CIDR for IPv4 is not allowed
- No Transitive VPC Peering
- No Edge to Edge Routing

### VPC Endpoint Gateway

- Only works for S3 and DynamoDB, must create one gateway per VPC
- Must update route table entries
- Gateway is defined at the VPC level
- DNS resolution must be enabled in the VPC

### VPC Endpoint Interface

- Provision an ENI that will have a private endpoint interface hostname
- Leverage Security Groups for security
- Private DNS
- Interface can be accessed from Direct Connect and Site-to-Site VPN

### VPC Endpoint Services (PrivateLink)

- ENI with NLB, which can be used to connect VPC outside of the account

### Site to Site VPN

- On-premise:
    - Setup a software or hardware VPN appliance to your on-premise network
    - The on-premise VPN should be accessible using a public IP
- AWS:
    - Setup a **Virtual Private Gateway** and attach to your VPC
    - Setup a Customer Gateway to point the on-premise VPN appliance
- Two VPN connections are created for redundancy, encrypted using IPSec

### AWS VPN CloudHub

- Can connect up to 10 Customer Gateway for each Virtual Private Gateway
- Low cost hub-and-spoke model for primary or secondary network connectivity between locations
- Provide secure communication between sites
- It's a VPN connection so it goes over the public internet
- Can be a failover connection between your on-premise locations

### Direct Connect

- Not something can be done immediately
- Provides a dedicate private connection from a remote network to the VPC
- Dedicated connection must be setup between your DC and AWS Direct Connect locations
- More expensive than running a VPN solution
- Bypass ISP, reduce network cost, increate bandwidth and stability
- Not redundant by default (must setup a failover DX or VPN)
- Data in transit is not encrypted but is private

### Direct Connect VIF

- VIF = Virtual Interface
- Public VIF: Connect to Public AWS Endpoints
- Private VIF: TO connect to resources in your VPC
- VPC endpoints cannot be accessed through Private VIF

### Direct Connect Gateways

- Direct Connect to one or more VPC in many different regions must use a Direct Connect Gateway
