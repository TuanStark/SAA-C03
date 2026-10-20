# AWS CERTIFIED SOLUTIONS ARCHITECT - ASSOCIATE (SAA-C03)
## DIAGNOSTIC FULL PRACTICE EXAM (65 QUESTIONS - 130 MINUTES)
*Target Score: $\ge$ 52/65 (80%+)*

---

### SECTION 1: DOMAIN 1 - DESIGN SECURE ARCHITECTURES (QUESTIONS 1 - 20)

#### QUESTION 1
A financial company must store customer tax records on Amazon S3 for 7 years. Regulatory compliance mandates that once written, these records must never be modified or deleted by anyone, including the AWS account root user, throughout the 7-year retention period. Which solution meets these requirements with the LEAST operational overhead?
- **A.** Enable S3 Versioning and configure MFA Delete on the bucket.
- **B.** Enable S3 Object Lock on the bucket in Compliance mode with a retention period of 7 years.
- **C.** Enable S3 Object Lock on the bucket in Governance mode with a retention period of 7 years.
- **D.** Configure an S3 Bucket Policy with an explicit Deny for all `s3:Delete*` actions.

#### QUESTION 2
A web application running on Amazon EC2 instances connects to an Amazon RDS for MySQL database. Security policy requires that database credentials must be automatically rotated every 30 days without downtime and without requiring manual intervention. Which solution should a solutions architect recommend?
- **A.** Store the credentials in AWS Systems Manager Parameter Store as a `SecureString`.
- **B.** Store the credentials in AWS Secrets Manager and configure Automatic Rotation with an AWS Lambda rotation function.
- **C.** Store the credentials in an Amazon S3 bucket encrypted with an AWS KMS Customer Managed Key (CMK).
- **D.** Use Amazon Cognito User Pools to manage database user authentication.

#### QUESTION 3
A multi-national enterprise uses AWS Organizations to manage multiple member accounts. A security administrator must ensure that no member accounts can create resources outside of two approved regions: `ap-southeast-1` and `us-east-1`. However, administrators must still be able to access the AWS Management Console and manage global services such as IAM, Route 53, and CloudFront. Which solution meets these requirements?
- **A.** Attach a Service Control Policy (SCP) to the organization root with a Deny effect for `aws:RequestedRegion`, using the `NotAction` element to exempt global services (IAM, Route 53, CloudFront, STS).
- **B.** Attach an IAM Policy to all IAM Users in each member account denying actions outside the approved regions.
- **C.** Configure Network ACLs in all VPCs to deny inbound and outbound traffic to non-approved regions.
- **D.** Create separate AWS Organizations for each approved region.

#### QUESTION 4
An application running on an EC2 instance in Account A needs to read analytics files from an Amazon S3 bucket located in Account B. To adhere to security best practices, no long-term IAM access keys or credentials should be stored on the EC2 instance or in application code. Which configuration achieves this?
- **A.** Create an IAM Role in Account B granting read access to the S3 bucket with a Trust Policy allowing Account A to assume it. Attach an IAM Role (Instance Profile) to the EC2 instance in Account A granting permission to call `sts:AssumeRole` on the Account B role.
- **B.** Make the S3 bucket in Account B public and restrict access to the Elastic IP of the EC2 instance.
- **C.** Store the AWS Access Key and Secret Key of Account B inside AWS Secrets Manager in Account A.
- **D.** Establish a VPC Peering connection between Account A and Account B and use an S3 Gateway Endpoint.

#### QUESTION 5
A company is hiring a third-party SaaS vendor to analyze AWS spending. The SaaS vendor provides a multi-tenant platform and requires access to the company's AWS account by assuming an IAM role via `sts:AssumeRole`. To prevent the "Confused Deputy" vulnerability, what should the solutions architect configure?
- **A.** Require the vendor to sign in using the AWS root account.
- **B.** Require the vendor to provide an External ID and add an `sts:ExternalId` condition to the IAM role's Trust Policy.
- **C.** Restrict access to the vendor's static IP range in the IAM policy.
- **D.** Embed long-term IAM credentials in the vendor's software.

#### QUESTION 6
A company wants to allow developers to create new IAM roles for AWS Lambda functions and Amazon EC2 instances. However, the security team must prevent developers from escalating their privileges or granting full `AdministratorAccess` to newly created roles. Which solution satisfies these requirements?
- **A.** Define a Permissions Boundary that limits maximum allowable permissions and require developers to attach this boundary to any new IAM role they create.
- **B.** Attach an AWS Organizations Service Control Policy (SCP) directly to the developers' IAM group.
- **C.** Enable Amazon GuardDuty to automatically remediate privilege escalation.
- **D.** Restrict developers to using only AWS CloudFormation to deploy resources.

#### QUESTION 7
A company wants to distribute its website globally using Amazon CloudFront with a custom domain name (`https://www.example.com`). The company wants to use a free SSL/TLS certificate provided by AWS Certificate Manager (ACM). In which AWS Region must the certificate be requested or imported?
- **A.** In the same Region as the Origin server.
- **B.** In `us-east-1` (N. Virginia).
- **C.** In `eu-west-1` (Ireland).
- **D.** In any Region where ACM is available.

#### QUESTION 8
A ticketing website experiences malicious bot scraping and credential-stuffing attacks during high-demand concert sales. Specific IP addresses are generating more than 2,000 requests every 5 minutes, overwhelming the Application Load Balancer. Which AWS WAF feature automatically detects and mitigates this traffic at the edge?
- **A.** Rate-based rule.
- **B.** SQL injection match condition.
- **C.** Geo-match rule.
- **D.** Byte match rule.

#### QUESTION 9
An insurance company stores millions of customer claims, scanned IDs, and credit card numbers across numerous Amazon S3 buckets. To maintain compliance, the company needs an automated service that uses machine learning to discover, classify, and protect sensitive data (PII) stored in S3. Which service should they use?
- **A.** Amazon GuardDuty
- **B.** Amazon Macie
- **C.** AWS CloudTrail
- **D.** Amazon Inspector

#### QUESTION 10
A solutions architect must provide secure shell (CLI) access to an EC2 instance running in a private subnet. The instance does not have a public IPv4 address, and security rules strictly forbid opening port 22 (SSH) on the Security Group or deploying bastion hosts. Which solution should the architect implement?
- **A.** Install the SSM Agent on the instance, attach the `AmazonSSMManagedInstanceCore` policy to its IAM role, configure Interface VPC Endpoints for Systems Manager, and use AWS Systems Manager Session Manager.
- **B.** Establish an AWS Site-to-Site VPN and open port 22 for the on-premises CIDR block.
- **C.** Assign an Elastic IP to the instance temporarily when access is required.
- **D.** Use AWS Cloud9 in a public subnet to connect to the private instance.

#### QUESTION 11
An enterprise policy mandates that all Amazon EBS volumes created in an AWS account must be encrypted at rest. If an unencrypted volume is detected, the system must automatically flag it and trigger automated remediation. Which architecture should be deployed?
- **A.** Deploy an AWS Config managed rule (`encrypted-volumes`) with Automatic Remediation linked to an AWS Systems Manager Automation document.
- **B.** Set up a CloudWatch Metric Alarm based on EBS IOPS.
- **C.** Write a cron job on an EC2 instance that runs hourly to scan for unencrypted volumes.
- **D.** Use Amazon Inspector to scan block storage daily.

#### QUESTION 12
A company needs to encrypt 100 GB database backup files using AWS KMS before uploading them to Amazon S3. The development team reports that calling the `kms:Encrypt` API fails due to size limits. Which standard cryptographic pattern solves this problem?
- **A.** Envelope Encryption using the `kms:GenerateDataKey` API to obtain a plaintext Data Key to encrypt the file locally, then storing the encrypted Data Key alongside the ciphertext.
- **B.** Compressing the backup file to under 4 KB using gzip before calling `kms:Encrypt`.
- **C.** Uploading the backup file to Amazon DynamoDB first.
- **D.** Using S3 SSE-C with customer-managed keys.

#### QUESTION 13
A compliance officer requires all data in transit between clients and an Amazon S3 bucket to be encrypted using HTTPS (TLS). Any insecure HTTP requests must be rejected immediately. Which Condition in an S3 Bucket Policy enforces this?
- **A.** `"Effect": "Deny"`, Condition: `{"Bool": {"aws:SecureTransport": "false"}}`
- **B.** `"Effect": "Allow"`, Condition: `{"Bool": {"aws:SecureTransport": "true"}}`
- **C.** `"Effect": "Deny"`, Condition: `{"StringEquals": {"s3:x-amz-acl": "public-read"}}`
- **D.** Enable default SSE-S3 encryption on the bucket.

#### QUESTION 14
A security auditor suspects an employee downloaded sensitive customer files (`s3:GetObject`) from a private S3 bucket before leaving the company. When inspecting the CloudTrail Event History, the auditor finds no records of object downloads. What must be configured to log object-level operations in the future?
- **A.** Enable Data Events for S3 buckets within an AWS CloudTrail trail.
- **B.** Upgrade to AWS Enterprise Support.
- **C.** Enable CloudWatch Detailed Monitoring on the S3 bucket.
- **D.** Configure S3 Object Lock on the bucket.

#### QUESTION 15
To prove to external auditors that CloudTrail audit log files stored in an S3 bucket have not been modified, overwritten, or deleted by an adversary, which feature should be enabled?
- **A.** CloudTrail Insights.
- **B.** CloudTrail Log File Integrity Validation.
- **C.** S3 Intelligent-Tiering.
- **D.** AWS KMS default key encryption.

#### QUESTION 16
An application experiences a volumetric Layer 3/4 Distributed Denial of Service (DDoS) attack targeting Route 53 DNS and CloudFront endpoints. Which service provides automatic, always-on protection against these infrastructure attacks at no additional cost?
- **A.** AWS Shield Standard
- **B.** AWS Shield Advanced
- **C.** AWS WAF
- **D.** Amazon GuardDuty

#### QUESTION 17
A company needs to store application configuration variables and database URLs in a hierarchical structure (`/app/prod/db_url`). The solution must be completely free of charge and integrate natively with AWS services. Which service should be used?
- **A.** AWS Systems Manager Parameter Store (Standard Tier)
- **B.** AWS Secrets Manager
- **C.** Amazon DynamoDB
- **D.** Amazon S3 Standard

#### QUESTION 18
To prevent Server-Side Request Forgery (SSRF) attacks from extracting temporary IAM credentials from the EC2 Instance Metadata Service, what should be configured across all EC2 instances?
- **A.** Enforce IMDSv2 by setting `HttpTokens=required`.
- **B.** Attach a second Elastic Network Interface (ENI).
- **C.** Block outbound port 443 in the Security Group.
- **D.** Assign an IPv6 address to each instance.

#### QUESTION 19
Which architectural challenge is resolved by using an AWS KMS Multi-Region Key?
- **A.** Encrypting data in one AWS Region and decrypting it locally in another Region without making cross-region API calls.
- **B.** Automatically shifting S3 storage tiers between Regions.
- **C.** Balancing database read/write queries across continents.
- **D.** Sharing VPC Peering connections across multiple accounts.

#### QUESTION 20
An IAM user has an identity-based policy granting `s3:GetObject` on all buckets. However, an AWS Organizations Service Control Policy (SCP) attached to the account has a statement with `"Effect": "Deny"`, `"Action": "s3:*"`. What is the result when the user tries to download an S3 object?
- **A.** Access is denied because an explicit Deny overrides any Allow.
- **B.** Access is allowed because identity-based policies take precedence over SCPs.
- **C.** Access is allowed only for buckets created before the SCP was attached.
- **D.** Access is granted temporarily for 15 minutes.

---

### SECTION 2: DOMAIN 2 - DESIGN RESILIENT ARCHITECTURES (QUESTIONS 21 - 37)

#### QUESTION 21
A two-tier web application requires high availability for its relational database layer. If the primary Availability Zone experiences an outage, the database must fail over automatically to a standby instance in another AZ without manual intervention and without changing the application's connection string. Which solution should be deployed?
- **A.** Amazon RDS for MySQL in a Multi-AZ Deployment.
- **B.** Amazon RDS for MySQL with a Read Replica in the same AZ.
- **C.** Two EC2 instances running MySQL with cron-based replication.
- **D.** AWS Database Migration Service (AWS DMS) in continuous replication mode.

#### QUESTION 22
A real-time multiplayer mobile gaming application uses the UDP protocol. The backend runs on EC2 instances in Singapore and Frankfurt. The architecture requires gamers to connect with the lowest possible latency, and traffic must fail over automatically between Regions in under 1 minute if a Region becomes unhealthy. Which solution is best?
- **A.** AWS Global Accelerator with Endpoint Groups in both Regions.
- **B.** Amazon CloudFront Distribution with Origin Shield.
- **C.** Route 53 Weighted Routing with CNAME records.
- **D.** Application Load Balancer with HTTP/2 enabled.

#### QUESTION 23
When an e-commerce order is submitted, the event must be processed concurrently by three independent microservices: Billing, Inventory, and Notifications. If the Inventory microservice experiences downtime or high latency, the Billing and Notification services must continue processing orders without delay or data loss. Which architecture fulfills this requirement?
- **A.** Publish order events to an Amazon SNS Topic, and subscribe three separate Amazon SQS Queues to the topic (Fan-Out Pattern), allowing each microservice to pull from its own queue.
- **B.** Have the front-end send synchronous HTTP POST requests directly to all three microservices.
- **C.** Have all three microservices read from a single shared Amazon SQS Standard Queue.
- **D.** Use AWS Batch to process orders sequentially.

#### QUESTION 24
A global financial application requires an RPO of less than 1 second and an RTO of less than 1 minute for a relational database across two AWS Regions (Singapore and Tokyo). Which solution meets these recovery objectives?
- **A.** Amazon Aurora Global Database.
- **B.** Amazon RDS for MySQL with a Cross-Region Read Replica.
- **C.** Hourly automated snapshots copied to Tokyo using AWS Backup.
- **D.** Amazon DynamoDB with DynamoDB Streams.

#### QUESTION 25
An enterprise wants to deploy a Disaster Recovery strategy to a secondary AWS Region using the **Warm Standby** approach. What defines this strategy?
- **A.** A scaled-down version of the fully functional production environment runs 24/7 in the secondary Region, ready to scale out to full capacity upon failover.
- **B.** No compute instances are running in the secondary Region; compute is provisioned only after disaster strikes.
- **C.** Full 100% capacity runs actively in both Regions concurrently.
- **D.** Servers in the secondary Region are turned off and manually booted by operators.

#### QUESTION 26
An online brokerage processes stock purchase requests using an Amazon SQS FIFO Queue. Orders from the *same customer* must be processed strictly in the order they were submitted, but orders from *different customers* should be processed concurrently in parallel by multiple worker instances. Which message attribute configuration achieves this?
- **A.** Set the `MessageGroupId` to the unique `CustomerID`.
- **B.** Set the `MessageDeduplicationId` to the `CustomerID`.
- **C.** Set the Visibility Timeout to 12 hours.
- **D.** Enable Long Polling with `WaitTimeSeconds=20`.

#### QUESTION 27
An image processing worker pulls messages from an Amazon SQS Standard queue. Processing an image takes 3 minutes. However, after approximately 30 seconds of processing, another worker pulls the same message and begins duplicate processing. What is the root cause and the appropriate solution?
- **A.** The processing time exceeds the default SQS Visibility Timeout (30 seconds). Increase the queue's Visibility Timeout or call `ChangeMessageVisibility` periodically from the worker.
- **B.** The SQS queue is corrupted and must be recreated.
- **C.** The worker instance is running out of memory.
- **D.** SQS Standard queues do not support media processing.

#### QUESTION 28
A global airline booking platform requires a database that allows users in both Tokyo and London to read and write data locally with single-digit millisecond latency (Active-Active Multi-Region read and write). Which database engine provides this capability natively?
- **A.** Amazon DynamoDB Global Tables.
- **B.** Amazon RDS PostgreSQL Multi-AZ.
- **C.** Amazon Aurora Serverless v2.
- **D.** Amazon Redshift.

#### QUESTION 29
A healthcare company wants to ensure that backup archives stored in AWS Backup cannot be deleted, altered, or shortened in retention period by anyone—including the root account—even if an administrator's credentials are compromised by ransomware. What should be enabled?
- **A.** AWS Backup Vault Lock in Compliance mode.
- **B.** S3 Versioning on backup buckets.
- **C.** AWS WAF Rate-based rules.
- **D.** AWS Shield Standard.

#### QUESTION 30
A company wants to replicate hundreds of on-premises VMware virtual machines to AWS for disaster recovery. The company wants to minimize ongoing AWS costs by avoiding running expensive standby EC2 instances during normal operations, but needs an RTO of minutes during a site outage. Which service should be used?
- **A.** AWS Elastic Disaster Recovery (AWS DRS)
- **B.** AWS DataSync
- **C.** AWS Database Migration Service (AWS DMS)
- **D.** AWS Snowball Edge

#### QUESTION 31
A batch processing worker fleet on EC2 needs to scale dynamically based on the volume of unprocessed tasks waiting in an Amazon SQS queue. Which Amazon CloudWatch metric should drive the Auto Scaling Group policy?
- **A.** `ApproximateNumberOfMessagesVisible`
- **B.** `CPUUtilization` on the Application Load Balancer
- **C.** `NetworkIn` on the NAT Gateway
- **D.** `VolumeWriteOps` on the EBS volume

#### QUESTION 32
A startup wants to route traffic for its apex domain (`example.com`) directly to an Application Load Balancer with high availability and without incurring DNS query charges. Which Route 53 record configuration is correct?
- **A.** Create an A (ALIAS) record pointing to the Application Load Balancer.
- **B.** Create a CNAME record pointing to the ALB DNS name.
- **C.** Create a PTR record.
- **D.** Create an MX record.

#### QUESTION 33
In a **Pilot Light** disaster recovery architecture, how is compute capacity managed in the secondary Region during normal operations to optimize cost?
- **A.** Set the Auto Scaling Group `DesiredCapacity=0` and `MinSize=0` in the secondary Region, keeping AMIs ready.
- **B.** Purchase 3-year Reserved Instances for all secondary servers.
- **C.** Run 100% of compute instances using Spot Instances.
- **D.** Use an infinite while loop in AWS Lambda.

#### QUESTION 34
An Auto Scaling Group spans two Availability Zones. When a scale-in event occurs, which instance will the default termination policy select for removal first?
- **A.** An instance in the Availability Zone with the most instances that has the oldest launch template.
- **B.** The instance with the lowest private IP address.
- **C.** The instance with the highest CPU utilization.
- **D.** The instance that was most recently launched.

#### QUESTION 35
An application running on EC2 instances needs to flush log files from local ephemeral storage to an Amazon S3 bucket before the instance is terminated during an Auto Scaling scale-in event. Which mechanism allows this graceful shutdown?
- **A.** Configure an Auto Scaling Lifecycle Hook for the `EC2_INSTANCE_TERMINATING` transition.
- **B.** Create a CloudWatch Metric Filter.
- **C.** Increase the default Cooldown Period to 1,000 seconds.
- **D.** Disable Termination Protection.

#### QUESTION 36
A company needs to orchestrate a serverless microservices workflow that includes automated tasks as well as a human approval step that can take up to 5 days to complete. Which service is designed for this?
- **A.** AWS Step Functions (Standard Workflow) with a Task Token callback.
- **B.** AWS Step Functions (Express Workflow).
- **C.** AWS Lambda with an execution timeout of 7 days.
- **D.** Amazon SQS Standard Queue.

#### QUESTION 37
An Amazon Aurora MySQL cluster has 1 Primary writer instance and 3 Aurora Read Replicas. Which DNS endpoint should be used by reporting applications to load balance read queries across all available read replicas?
- **A.** Reader Endpoint
- **B.** Cluster Endpoint
- **C.** Instance Endpoint
- **D.** Gateway Endpoint

---

### SECTION 3: DOMAIN 3 - DESIGN HIGH-PERFORMING ARCHITECTURES (QUESTIONS 38 - 52)

#### QUESTION 38
A tightly coupled High-Performance Computing (HPC) workload requires ultra-low network latency (sub-millisecond) and high throughput (up to 100 Gbps) between dozens of EC2 instances. Which EC2 placement group should be selected?
- **A.** Cluster Placement Group within a single Availability Zone.
- **B.** Spread Placement Group across multiple AZs.
- **C.** Partition Placement Group.
- **D.** Multi-Region Placement Group.

#### QUESTION 39
A mission-critical enterprise relational database requires predictable block storage performance delivering over 100,000 IOPS with sub-millisecond latency. Which Amazon EBS volume type should be selected?
- **A.** Provisioned IOPS SSD (`io2 Block Express`)
- **B.** General Purpose SSD (`gp3`)
- **C.** Throughput Optimized HDD (`st1`)
- **D.** Cold HDD (`sc1`)

#### QUESTION 40
A real-time 3D physics rendering engine on EC2 requires temporary scratch storage capable of delivering hundreds of thousands of IOPS with microsecond latency. The data does not need to persist when the instance is stopped or terminated. Which storage option offers the highest performance at the lowest cost?
- **A.** EC2 Instance Store (attached NVMe SSD).
- **B.** Amazon EBS `io2 Block Express`.
- **C.** Amazon EFS in Provisioned Throughput mode.
- **D.** Amazon S3 Transfer Acceleration.

#### QUESTION 41
A fleet of Linux-based web servers deployed across three Availability Zones requires concurrent read/write access to a shared directory compatible with standard POSIX file permissions. Which storage service fits this requirement?
- **A.** Amazon EFS (Elastic File System)
- **B.** Amazon EBS with Multi-Attach enabled
- **C.** Amazon FSx for Windows File Server
- **D.** Amazon S3 Glacier

#### QUESTION 42
An e-commerce company wants to reduce read latency on popular product catalog items in Amazon DynamoDB from milliseconds to **microseconds** without rewriting complex application logic. Which solution should they deploy?
- **A.** Amazon DynamoDB Accelerator (DAX)
- **B.** Amazon ElastiCache for Memcached
- **C.** Double the Read Capacity Units (RCU)
- **D.** DynamoDB Global Tables

#### QUESTION 43
A mobile gaming company needs to build a real-time global leaderboard that tracks the top 100 players ranked by score. The caching layer must support the **Sorted Sets** data structure and provide Multi-AZ automated failover. Which service should be chosen?
- **A.** Amazon ElastiCache for Redis
- **B.** Amazon ElastiCache for Memcached
- **C.** Amazon RDS PostgreSQL
- **D.** Amazon DynamoDB Standard

#### QUESTION 44
A machine learning training pipeline requires a distributed file system that can process massive training datasets with sub-millisecond latencies and integrates natively with an Amazon S3 data lake for bidirectional synchronization. Which file system should be selected?
- **A.** Amazon FSx for Lustre
- **B.** Amazon FSx for Windows File Server
- **C.** Amazon EFS
- **D.** AWS Storage Gateway

#### QUESTION 45
A media company delivers high-resolution images and videos to a global audience. The company wants to reduce website load times and offload origin traffic from their Amazon S3 bucket by caching static content close to users worldwide. Which service should they implement?
- **A.** Amazon CloudFront
- **B.** AWS Global Accelerator
- **C.** Amazon Route 53
- **D.** AWS Transit Gateway

#### QUESTION 46
A financial payment API built on AWS Lambda experiences 3 to 5 seconds of latency during the first invocation of the day due to container initialization (Cold Start). Which Lambda feature eliminates cold starts entirely for latency-critical workloads?
- **A.** Provisioned Concurrency
- **B.** Reserved Concurrency
- **C.** Allocating 10 GB of RAM
- **D.** Migrating to HTTP API

#### QUESTION 47
A financial trading platform processes tens of millions of requests per second requiring sub-millisecond latency at Layer 4, needs to preserve client IP addresses natively, and requires static IP addresses per Availability Zone for partner firewall whitelisting. Which load balancer should be chosen?
- **A.** Network Load Balancer (NLB)
- **B.** Application Load Balancer (ALB)
- **C.** Gateway Load Balancer (GLB)
- **D.** Classic Load Balancer

#### QUESTION 48
A company connects its headquarters to AWS via a dedicated 10 Gbps AWS Direct Connect connection. To connect this single physical connection to multiple VPCs across different AWS Regions through a centralized hub, which component should be configured?
- **A.** Direct Connect Gateway
- **B.** Internet Gateway
- **C.** NAT Gateway
- **D.** VPC Peering

#### QUESTION 49
To optimize upload speeds for users worldwide uploading multi-gigabyte video files to a centralized Amazon S3 bucket using Amazon's globally distributed Edge Locations, which feature should be enabled?
- **A.** S3 Transfer Acceleration
- **B.** S3 Multi-Region Access Points
- **C.** S3 Select
- **D.** S3 Lifecycle Rules

#### QUESTION 50
A data analyst needs to query specific rows and columns from multi-gigabyte CSV files stored in an Amazon S3 bucket using simple SQL expressions without downloading the entire file to an EC2 instance. Which feature should be used?
- **A.** S3 Select
- **B.** S3 Glacier Deep Archive
- **C.** S3 Object Lock
- **D.** S3 Batch Operations

#### QUESTION 51
A company wants to route global end users to the AWS Region that provides the lowest network latency. Which Amazon Route 53 routing policy should be selected?
- **A.** Latency-based Routing
- **B.** Geolocation Routing
- **C.** Weighted Routing
- **D.** Failover Routing

#### QUESTION 52
An Amazon RDS for PostgreSQL database experiences 95% CPU utilization due to a high volume of read-only queries (`SELECT`) generated by business intelligence dashboards. Which architectural modification offloads read traffic with the least overhead?
- **A.** Deploy RDS Read Replicas and point the reporting dashboards to the Read Replica endpoints.
- **B.** Convert the database to a Multi-AZ Deployment.
- **C.** Increase the allocated EBS storage volume size.
- **D.** Deploy additional NAT Gateways in the VPC.

---

### SECTION 4: DOMAIN 4 - DESIGN COST-OPTIMIZED ARCHITECTURES (QUESTIONS 53 - 65)

#### QUESTION 53
A genomics research company runs batch analysis jobs overnight using an Amazon SQS queue to distribute tasks to worker nodes. The jobs are stateless, checkpoint data regularly to S3, and can tolerate unexpected interruptions. Which EC2 purchasing option provides up to 90% cost savings?
- **A.** Spot Instances
- **B.** On-Demand Instances
- **C.** 3-year Standard Reserved Instances
- **D.** Dedicated Hosts

#### QUESTION 54
An enterprise enterprise resource planning (ERP) system runs on EC2 instances 24/7/365 with steady-state utilization expected to remain constant over the next 3 years. Which pricing model yields the highest cost savings?
- **A.** 3-year Standard Reserved Instances (or EC2 Instance Savings Plans) with All Upfront payment.
- **B.** On-Demand Instances with per-second billing.
- **C.** Spot Instances.
- **D.** Scheduled Reserved Instances.

#### QUESTION 55
A company stores hundreds of terabytes of files on Amazon S3 with unpredictable access patterns. Some files are accessed frequently, while others remain untouched for months before sudden spikes in access. The company wants to minimize storage costs automatically with no retrieval fees and no operational management. Which S3 storage class is best?
- **A.** S3 Intelligent-Tiering
- **B.** S3 Standard-IA
- **C.** S3 Glacier Flexible Retrieval
- **D.** S3 One Zone-IA

#### QUESTION 56
A corporation is required by financial law to archive audit log files for 10 years. The files are rarely retrieved except in the event of an audit, and retrieval times of 12 to 48 hours are acceptable. Which S3 storage class provides the lowest storage cost across all of AWS?
- **A.** S3 Glacier Deep Archive
- **B.** S3 Glacier Instant Retrieval
- **C.** S3 Standard-IA
- **D.** S3 One Zone-IA

#### QUESTION 57
EC2 instances in a private subnet frequently upload large analytical datasets to an Amazon S3 bucket. Currently, all traffic routes through a NAT Gateway, incurring high data processing charges. What is the most cost-effective way to route this traffic?
- **A.** Create an S3 Gateway Endpoint in the VPC and update the route table (free of charge).
- **B.** Move the EC2 instances to a public subnet and assign Elastic IPs.
- **C.** Deploy an S3 Interface Endpoint (PrivateLink).
- **D.** Establish an AWS Site-to-Site VPN.

#### QUESTION 58
A high-throughput application performs hundreds of thousands of read requests per second on an S3 bucket encrypted with SSE-KMS, generating high AWS KMS API costs and risking KMS request throttling. Which feature reduces KMS API requests by up to 99%?
- **A.** Enable S3 Bucket Keys.
- **B.** Switch to SSE-C encryption.
- **C.** Generate 10 separate KMS keys.
- **D.** Move the data to EBS volumes.

#### QUESTION 59
A media company stores files on an Amazon EFS file system. Analysis shows that 80% of files are rarely accessed after 30 days. Which feature automatically moves cold files to a storage tier that is up to 90% cheaper without disrupting user access?
- **A.** EFS Lifecycle Management configured to transition files to EFS Infrequent Access (EFS-IA).
- **B.** S3 Lifecycle Rules.
- **C.** AWS Backup Vault Lock.
- **D.** Writing an EC2 cron job to delete old files.

#### QUESTION 60
A development team needs to build a lightweight API that acts as a pass-through proxy to invoke AWS Lambda functions, supports OpenID Connect authentication, and costs up to 70% less than a REST API. Which API Gateway type should be chosen?
- **A.** HTTP API
- **B.** REST API Edge-Optimized
- **C.** WebSocket API
- **D.** Private REST API

#### QUESTION 61
A company wants to configure an Auto Scaling Group that maintains a baseline of On-Demand instances to ensure availability, and scales out using Spot Instances to handle variable traffic spikes at the lowest possible cost. Which feature allows this configuration within a single ASG?
- **A.** Mixed Instances Policy
- **B.** Target Tracking Scaling Policy
- **C.** Step Scaling Policy
- **D.** Cooldown Period

#### QUESTION 62
To optimize Amazon S3 storage costs when users upload large files using Multipart Uploads that sometimes get abandoned or interrupted before completion, which S3 Lifecycle rule should be configured?
- **A.** Abort incomplete multipart uploads after N days.
- **B.** Transition to S3 Glacier after 30 days.
- **C.** Delete expired object delete markers.
- **D.** Enable S3 Versioning.

#### QUESTION 63
A startup needs to run a relational MySQL database for an application with unpredictable and spiky traffic—busy for several hours during the day and virtually idle overnight. Which database solution automatically scales compute capacity in fractions of a second to minimize cost?
- **A.** Amazon Aurora Serverless v2
- **B.** Amazon RDS MySQL Multi-AZ
- **C.** Amazon Redshift
- **D.** Amazon DocumentDB

#### QUESTION 64
An application stores temporary shopping cart session records in an Amazon DynamoDB table. The records must be deleted automatically after 24 hours without consuming Write Capacity Units (WCU) or running batch delete scripts. Which DynamoDB feature satisfies this?
- **A.** DynamoDB Time to Live (TTL)
- **B.** DynamoDB Streams
- **C.** DynamoDB Global Tables
- **D.** DynamoDB Transactions

#### QUESTION 65
When connecting two VPCs in the same AWS Region, which networking option provides high bandwidth, low latency, and incurs **no ongoing hourly connection fee**?
- **A.** VPC Peering
- **B.** AWS Transit Gateway
- **C.** AWS Site-to-Site VPN
- **D.** NAT Gateway

---

---

## OFFICIAL ANSWER KEY & DOMAIN CLASSIFICATION

| Question | Answer | Domain | Question | Answer | Domain | Question | Answer | Domain | Question | Answer | Domain |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | **B** | Domain 1 | **18** | **A** | Domain 1 | **35** | **A** | Domain 2 | **52** | **A** | Domain 3 |
| **2** | **B** | Domain 1 | **19** | **A** | Domain 1 | **36** | **A** | Domain 2 | **53** | **A** | Domain 4 |
| **3** | **A** | Domain 1 | **20** | **A** | Domain 1 | **37** | **A** | Domain 2 | **54** | **A** | Domain 4 |
| **4** | **A** | Domain 1 | **21** | **A** | Domain 2 | **38** | **A** | Domain 3 | **55** | **A** | Domain 4 |
| **5** | **B** | Domain 1 | **22** | **A** | Domain 2 | **39** | **A** | Domain 3 | **56** | **A** | Domain 4 |
| **6** | **A** | Domain 1 | **23** | **A** | Domain 2 | **40** | **A** | Domain 3 | **57** | **A** | Domain 4 |
| **7** | **B** | Domain 1 | **24** | **A** | Domain 2 | **41** | **A** | Domain 3 | **58** | **A** | Domain 4 |
| **8** | **A** | Domain 1 | **25** | **A** | Domain 2 | **42** | **A** | Domain 3 | **59** | **A** | Domain 4 |
| **9** | **B** | Domain 1 | **26** | **A** | Domain 2 | **43** | **A** | Domain 3 | **60** | **A** | Domain 4 |
| **10** | **A** | Domain 1 | **27** | **A** | Domain 2 | **44** | **A** | Domain 3 | **61** | **A** | Domain 4 |
| **11** | **A** | Domain 1 | **28** | **A** | Domain 2 | **45** | **A** | Domain 3 | **62** | **A** | Domain 4 |
| **12** | **A** | Domain 1 | **29** | **A** | Domain 2 | **46** | **A** | Domain 3 | **63** | **A** | Domain 4 |
| **13** | **A** | Domain 1 | **30** | **A** | Domain 2 | **47** | **A** | Domain 3 | **64** | **A** | Domain 4 |
| **14** | **A** | Domain 1 | **31** | **A** | Domain 2 | **48** | **A** | Domain 3 | **65** | **A** | Domain 4 |
| **15** | **B** | Domain 1 | **32** | **A** | Domain 2 | **49** | **A** | Domain 3 | | | |
| **16** | **A** | Domain 1 | **33** | **A** | Domain 2 | **50** | **A** | Domain 3 | | | |
| **17** | **A** | Domain 1 | **34** | **A** | Domain 2 | **51** | **A** | Domain 3 | | | |
