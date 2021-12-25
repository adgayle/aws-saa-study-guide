# AWS Solution Architect - Associate
## AWS Global Infrastructure
* An Availability Zone (AZ) is one or more discrete data centers, each with redundant power, networking and connectivity housed in separate facilities
* A Region is a physical location in the workd which consists of two or more Availability Zones (AZ)
* Edge Locations are endpoints for AWS which are used for content caching. Typically consists of CloudFront, Amazon's Content Delivery Network (CDN)

## IAM
### 101
* Manages access to users and resources in an AWS account
* Allows for shared access to the AWS account by multiple users
* Granular permissions
* Identity Federation (Active Directory,  Facebook, LinkedIn, etc.)
* Multi-factor authentication (enable at least for root account)
* Provide temporary access for users / devices and services where necessary
* Default access is No Access
* Roles are Global
* You can now attach / detach roles to running EC2 instances

### Authorization and Authentication
* Users, Groups, Roles, Policy Documents (JSON)
* Universal so not region independent
* Root account has complete access
* New users have no access
* Programmatic access via Access Key ID and Secret Access Key. Shared only at creation so save them for SDK, CLI and API use. Cannot be used for console access.
* Console access via user and password
* Multi-factor and password policy can be enabled
    * Always enabled MFA for root user
    * Users have to enabled their own MFA
* Use groups to organize access
* Roles can be assigned to users or groups
* IAM policies are JSON documents granting permissions to users to access resources
* Managed policies are created by AWS and cannot be edited
* Customer policies are created by the customer
* Inline policies are attached directly to the user

## Cognito
Decentralized managed authentication system for your mobile, desktop or web application
* User pools allows users to authenticate using an OAuth iDP (identity Provider) e.g. Google to give access to your web applications
    * Uses JWT to persist authentication
* Identity pools provide temporary AWS credentials to access services e.g. S3
* Cognito Sync can sync user data across multiple devices (based on SNS)
* Web Identity Federation exchanges identity and security information between th iDP and the application
* OIDC type of identity provider which uses OAuth
* SAML type of identity provider which is used for SSO (Single Sign-on)

## EC2
Amazon Elastic Compute Cloud reduces the time required to obtain and boot new server instances. Scale up and down quickly. Pay only for what you need.
* Termination protection is off by default
* Root EBS is removed automatically by default on termination
* An instance profile attaches a role to an instance and allows it to perform tasks against AWS devices based on access.
* Cannot encrypt customer volumes on AWS AMI must use a custom and encrypt on copy
### Pricing Options
* On Demand - pay a fixed rate by the hour no commitment
    * Good for low cost no upfront payment
    * Short term, spiky or unpredictable workloads that cannot be interrupted
    * Applications still in testing
* Reserved - provides a capacity reservation with a big discount over a period of typically 1 or 3 years
    * Predictable and steady usage applications
    * Can be converted to different instance types
        * Can only change to larger or price increase
* Spot - enables you to bid whatever price with even greater savings if your needs are flexible. Price is driven by market
    * Require applications with flexible start and end time
    * Must be interruptible
    * Lowest cost
    * Great for large capacity
    * Terminated when the market price goes above your bid price
    * When Amazon terminates then you are not charged for the partial hour
    * When you terminate the instance you are charged for the usage
* Dedicated - Your own hosts but comes at a premium price
    * Good for regulatory that does not support multi-tenant virtualization
* Savings Plan
    * You commit to using $X/hour of compute and this gets you a price lower than On Demand
    * Applies to a specific instance family
    * No region restrictions

### EBS Backed
* Persistent
* Attach and reattach (root requires outage)
* Data remains if volumes are stopped
* Default is to terminate if volumes are stopped
* You need a custom AMI to encrypt the root volume
* Volume must be in the same AZ as the EC2 instance
* To encrypt a root device you can take a snapshot and then copy it and select encrypt then create an AMI from it to boot EC2 instances in different regions
* Stop instance before taking a root volume snapshots to ensure consistency
* Cannot share encrypt snapshots
* Volumes restored from encrypted snapshots are also encrypted
### Instance Backed
* Not persistent (ephemeral)
* Not for long term data
* Data is deleted when terminated (no stop option)
* Cannot attach EBS volumes during launch only after launch
* Cannot be stopped
* If the virtualization host fails you VM and instance store data are lost
* Launched from templates stored in S3 so a little slower to launch
### Metadata
* curl http://169.254.169.254/meta-data
* Userdata can be used to bootstrap EC2 instances
    * To view the bootstrap script curl http://169.254.169.254/latest/user-data

### Placement Groups
* Clustered placement groups
    * Instance grouped in a single availability zone with low network latency and high throughput
    * Only certain instance types can be configured for this
    * Single AZ
* Spread placement groups
    * Instances placed on distinct underlying hardware
    * Used to separate instances of the same application on different hardware
    * Can span multiple AZs
    * Cannot be merged nor moved into a new placement group

## Amazon Machine Images (AMI)
Region specific templates from which to generate new EC2 instances
* Can be copied to other regions which generates unique IDs per region
* Can create from running or stopped instances
* AWS, community or marketplace provided
* Can be shared with other accounts or made public

## Auto Scaling Groups (ASG)
* Launch configuration specifies the AMI, Instance size, IAM role, bootstrap script, etc. Answers the question of what each EC2 instance launched should look like
* Autoscaling group specifies how many instances are launched, which network they are launched in and on which subnet (it applies this intelligently). Specifically a load balancer, health check and grace period. Also apply scaling policies beyond the initial settings for growning and shrinking. Notifications can be sent via SNS for launch, terminate and associated failures
* Launch configurations cannot be edited but you can clone
* Launch templates are versioned and can be used to create launch configurations
* Target scaling policy scales when the target value of a metric is breached
* Simple scaling policy when an alarm is breached
* Scaling policy with steps allows you to put a time interval or steps during which the alarm must be breached
* Desired capacity is how many instances you typically want in your ASG
* Health checks can be run against the EC2 instances or ELB (application checks)
* ASG uses the specified launch configuration when starting new EC2 instances

## EBS
Elastic Block Storage that is attached to EC2 instances
* Size can be changed on the fly
* Use RAID 0 to virtually increase IOPS limits
* Must be in the same AZ as the EC2 instance (latency issue)
* Move AZ/region using snapshot and copy
* Change to encrypted by snapshot and copy (set encryption flag)
* Volume Types
    * General Purpose (gp2 / gp3)
        * Bootable
        * Larger gp2 volumes have greater throughput
        * gp3 does not need a size increase to increase throughput (throughput independent of size)
        * Use for low latency apps with IOPs less than 16000
    * Provisioned IOPS (io1 / io2)
        * Bootable
        * I/O intensive applications e.g. NoSQL
        * Use for IOPs greater than 16000
    * Throughput Optimized (st1)
        * Not bootable
        * Log processing
        * Data warehousing
        * Big data
    * Cold HDD (sc1)
        * Lowest cost infrequently accessed
        * Not bootable
    * Magnetic (standard)
        * Low cost
        * Low performance
        * Bootable
        * Cannot be modified for performance
* Snapshots
    * Can copy across regions and AZs
    * Create AMI from a root disk snapshot to launch in new AZ or region
    * Point in time copy stored in S3
    * Only changed blocks (incremental) are stored in S3
    * Automatically encrypted if volumes are restored from encrypted volumes
    * Cannot share encrypted volumes (KMS key issue)
    * Application consistent snapshot options for RAID
        * Freeze the file system
        * Unmount the volume
        * Shutdown the EC2 instance

## CloudWatch
* EC2 Basic / Standard monitoring (5 minute interval / free tier)
* EC2 Detailed monitoring ( 1 minute / paid service)
* Can create dashboards and alarms (sent via SNS topics) which requires confirmation
* High resolution metrics allow you to have custom metrics under 1 minute as low as every 1 second
* CloudWatch metrics can be visualized in CloudWatch Dashboards
* CloudWatch EC2 metrics
    * CPU Usage
    * Disk Performance
    * Network Usage
    * Status Checks
* Memory usage and disk space utilization are not included by default and require CloudWatch Agent installation
* CloudWatch Events / Event Bridge enable you to respond to state changes in AWS resources
    * You can use lambda functions to respond to the state changes
* CloudWatch Logs used to aggregate, monitor and store logs
    * Requires an agent installed to send the logs
    * Stored indefinitely
    * Logs are stored in log groups
    * You can stream custom logs
* Use scripts or CloudWatch Agent to send custom metrics

## CloudTrail
Monitor API calls and actions made on an AWS account
* Per AWS account and per region (recommended to enabled in all)
* Consolidate to S3 for paying account or dedicated account for logs
* Can log all API calls to AWS (since everything uses an API call)
* Includes the source of the call, when, who and what was done
* Logs for 90 days in Event history
* Beyond 90 days store the logs in S3 and process with AWS Athena
* Can be enabled in multiple regions in a single account in single click
* Can be enabled in multiple accounts in an organization in a single click
* Logs can be encrypted via KMS
* Log file validation can be enabled to prevent tampering
* CloudTrail events can be sent to CloudWatch
* Data events are high volume and expensive to enable
* Management events are management operations performed in the account
* governance, compliance, operational auditin and risk auditing are keyword triggers

## Elastic Load Balancers (ELB)
Load Balancers spread load across resources to provide services. The ELB does not terminate unhealthy instances just stops traffic going to it if configured and the health check fails
* Cross zone load balancing allows you to distribute the workload across all instances evenly. When not enabled the workload is distributed evenly at the availability zone level
* Cannot span regions
* Supports SSL via ACM
* Autoscaling groups are associated to the ELB by means of the target group
* Request routing based on
    * Path
    * Header
    * Type of request e.g. GET
* Application Load Balancer (ALB)
    * Layer 7 (application aware)
    * Best suited for HTTP & HTTPS
    * Sticky sessions
    * Supports Web Access Firewall (WAF) attachment
* Network Load Balancer (NLB)
    * Best for TCP traffic where extreme performance is required
    * Can handle UDP traffic
    * Layer 4
    * Has no rules capability
* Classic Load Balancer (being deprecated)
    * Not recommended for use
    * Sticky sessions
        * Bind a user session to a specific instance because of session information
    * HTTP & HTTPS traffic
    * Not recommended for new deployments
    * 504 Gateway Timeout - means it is having a hard time reaching the down stream resource e.g. EC2 web application
    * X-Forward-For (XFF) header allows you to see the originating IP address of the person making the request
    * Use the DNS name as there is no public IP address released by AWS
    * Layer 4 & 7 but not as advanced as the ALB
    * Has no rules capability

## Elastic File System (EFS)
* Storage capacity shrinks and grows as needed
* Pay per GB used
* Available over NFSv4
* Can be shared across multiple EC2 instances in the same VPC
* Scale to petabytes
* Thousands of concurrent connections
* Block based storage
* Read after write consistency
* EC2 security group must allow NFS access to the EFS security group to mount the shared volume

## Lambda
Compute service where you upload your code ad create functions as a service. Handles scaling, operating systems, patching, etc. This is event driven or in response to HTTP requests
* Lambda Languages
    * C#, Node.js, Go, Java, Python, Ruby
* Lambda Triggers
    * API Gateway, IoT, Alexa Skills, Alexa Smart Home, CloudFront, CloudWatch Events, CloudWatch Logs, CodeCommit, Kinesis, Cognito Sync, S3, SNS, DynamoDB
* Each call invokes a new Lambda function (1 event = 1 function)
* Lambda functions can trigger another Lambda function(s)
* Pricing
    * Number of requests
    * Duration of execution
* Function cannot run for more than 15 minutes
* No servers
* Continuous scaling out
* Super cheap

## AWS CLI and SDK
AWS SDK provides API libraries for integrating AWS services into your applications
* Programmatic access must be enabled
* Use `aws configure` to setup CLI environment profiles in `~/.aws/credentials`
* SDK available for the following:
    * C++, Go, Java, Javascript, .NET, NodeJS, PHP, Python, Ruby

## VPC
Logical datacenter in AWS
* 1 subnet = 1 availability zone
* Security groups are stateful
* Security groups do not span VPCs
* One internet gateway per VPC
* Configure route tables between subnets
* Launch EC2 into subnet of choice

### Network Access Control Lists (NACL)
* New NACL default is DENY everything
* Default NACL allows everything
* Can use one NACL for multiple subnets
* Subnet can only have 1 NACL at a time and must have one, default is assigned if none assigned
* Rules are evaluted in numerical order (lowest first)
* Separate inbound and outbound rules
* Has DENY rule which 
* Network Access Control Lists (NACLs) are stateless (must open ports directly, remember ephemeral ports)
* Allow ephemeral ports for outbound
* Useful for blocking inbound traffic

### Security Groups
* Acts as a firewall at the instance level
* Up to 16 per ENI (default 5)
    * 60 inbound and 60 outbound rules per security group
    * 10000 security groups per region (default 2500)
* Can allow IP, IP range or another security group
* All outbound allowed by default
* All inbound blocked by default (no rule)
* Stateful (allowed inbound traffic is also allowed to return out)
* Changes are immediately effective
* There is no deny just allow
* Allows take precedence

### Default VPC
* All subnets are Internet Accessible
* EC2 instances have public and private address

### VPC flow logs 
* Allow you to capture the traffic sent and recieved by NIC within the VPC
* Stores the source and destination IP addresses
* Can be done at VPC, subnet or NIC level
* Cannot tag flow log
* Cannot enable flow logs on peer VPC unless it is in your account
* Cannot change the configuration of a flow log after 
* Can be sent to either S3 or CloudWatch Logs
* Amazon core infrastructure (DNS, DHCP, Windows license, metadata address, etc.) is not included

### VPC Peering
* Route traffic between VPC in same or different accounts
* No single point of failure or bandwidth limitations
* Cannot use the same or overlapping IP address (CIDR) block
* No transitive peering. Must be directly peered
* Cannot route direct connect of internet traffic. No edge routing

### Nat Instances
* Replaced by redundant NAT Gateway (paid AWS managed service per AZ)
* Must disable source and destination traffic checking so that it can forward traffic from other instances
* Must be in a public subnet
* Instance size if determined by traffic processed
* Must have a path from private subnet to NAT instance to get out

### Bastion Hosts / Jump Boxes
* Used to securely administer EC2 instances in private subnets via SSH / RDP
* Must be hardened
* Do not store private keys use SSH Forwarding

### VPC Endpoints
* Keep traffic for AWS services with the AWS network
* Two types
    * Interface endpoint
        * Costs money
        * Uses an Elastic Network Interface (ENI) with private IP (Powered by AWS PrivateLink)
        * Supports many AWS services
    * Gateway endpoint
        * Free
        * Route table entry
        * Supports with S3 and DynamoDB

## Direct Connect
Dedicated network line from on premise / co-location to AWS
* Reduces data costs
* Increases reliability
* Increases bandwidth
* 1GBps - 10GBps
* Takes time to implement the circuit

## S3
### 101
Key value based object store
* S3 is object-based storage allows you to upload and download files
* Files can be 0bytes - 5TB
* There is unlimited storage available
* Files are stored in buckets
* File names are the keys and the file data are the objects
* S3 bucket names are universal. That is they must be unique globally
* Link to file example https://s3-eu-west-1.amazonaws.com/mybucket-986
* New object creation is strongly consistent
* Deletes and overwrites of objects are eventually consistent

### Buckets
* Universal namespace
* Upload an object to S3 receive HTTP 200 code
* Types
    * S3 - Standard
        * Availability 99.9%
        * Durability 99.999999999% (11 9's)
    * S3 - Intelligent Tiering
        * Uses ML to anaaylze object use and apply the best storage class based on cost and access frequency 
    * S3 - Infrequent Access
        * Availability 99.9%
        * Durability 99.999999999% (11 9's)
        * Paid retrieval but quick
        * Cheap if you access less than once per month
        * 30 days old & minimum size charged 128KB to transfer from standard
    * S3 - One Zone Infrequent Access
        * Availability 99%
        * Durability 99.999999999% (11 9's)
        * Single AZ
        * Paid retrieval
* Control access to buckets with either bucket ACL or bucket policy (JSON granular)
* By default buckets are private and all objects stored inside them are private
* Delete markers are replicated
* Deleting individual versions or delete markers will not be replicated
* You cannot replicate to multiple buckets or use a daisy chain
* Presigned URL are created with the CLI and grant temporary timed access to an object or objects

### Glacier
#### S3 - Glacier
User to archive (backup) data with paid retrieval
* 30 days after IA or 1 day after S3
* Takes minutes to hours to retrieve data
* Retrieval types
    * Expedited
    * Standard
    * Bulk

#### S3 - Glacier Deep Archive
* Super cheap
* Retrieval time of 12 hours

### Cross Region Replication
* Versioning must be enabled on source and target
* Regions must be unique
* Files existing in the bucket are not replicated automatically just new or updated files

### Versioning
* Stores all version of an object (including all writes even if you delete the object)
* Great for backups
* If the current version is deleted the previous version is presented as the object
* You can apply permissions per version
* Once enabled, versioning cannot be disable, only suspended
* Integrates with lifecycle rules
* Versioning has multi-factor (MFA) integration to confirm deletes, providing additional security
    * Only the root account using the CLI can delete the object  with MFA enabled
    * MFA delete can only be enabled from the CLI

### Lifecycle Management
* Can be used in conjunction with versioning
* Can be applied to current and previous versions
* Following actions can be done immediately
    * Transition to Standard - Infrequent Access storage class after 30 days
    * Transition to Glacier storage class 30 days after Infrequent Access if relevant
    * Permanently delete

### Transfer Acceleration
* Uses CloudFront Edge locations to speed up file transfers with S3
* Uses AWS backbone to speed up upload once it reaches CloudFront
* Uses new URL e.g. bucketname.s3.accelerate.amazonaws.com

### Static Websites
* Policy to make the entire bucket public
* Websites that require database access cannot be hosted with S3 (unless you make calls to the API Gateway and Lamda)
* Scales automatically
* Sample URL bucketname.s3-websie-us-east-1.amazonaws.com (for website enabled)

### Security and Encryption
* New buckets are private
* Send access logs to separate S3 bucket for object requests
* Bucket policies are bucket wide vs. ACL which can go to the object level
* Encryption
    * Client side encryption
    * Server side encryption
        * Amazon S3 Managed Keys (SSE-S3 / SSE-AES256)
        * KMS (SSE-KMS)
        * Customer Provided Keys (SSE-C)
    * In Transit
        * SSL/TLS

## Security Token Service
Provides limited temporary access to AWS resources
* Via federation typicall Active Directory
* No IAM user needed
* Uses SAML
* Identity broker - Service that allows you to take identity from point A and federate or join it to point B
* Identity store - think AD, Facebook or Google login stores
* Identities - user of a service
Provides
* Access key
* Secret key
* Duration - time authentication is valid
* Token
Sequence of events
* User -> Application -> Identity Broker -> Identity Store (LDAP) -> AWS STS -> Application -> AWS Service -> AWS IAM -> Temporary access to AWS service

## CloudFront Content Delivery Network
Content distribution network (CDN) that makes web / video content load faster by providing a cache closer to the user
* Edge location is the location where the content is cached. Separate from regions and AZ
* Origin is all the files that the CDN will distribute. Can be S3 bucket, EC2 instances, ELB instance, Route53 or non aWS file store
* Distribution is the name given to a collection of Edge locations
* Edge locations can be written to (process uploads)
* Objects are cached for a TTL
* Cache can be cleared / invalidated but there is a charge
* Origin can be a folder in an S3 bucket, EC2, ELB
* S3 buckets do not have to be public and can limit access to just the distribution with OAI
* All HTTP methods are supported via the distribution including PUT and POST
* Restrict view access via using signed URLs and signed cookies
* Can further protect using WAF (Web Application Firewall)
* CNAMES can be specified
* Use Origin Access Identity (OAI) to access S3 origin and prevent everything else from accessing S3
    * Signed URLs
    * Signed Cookies
* SSL can be a custom certificate
* Content can be restricted by Geography
* Types of distributions
    * RTMP / RTP for video streaming
    * Static website hosting
* Lambda@Edge allows you to pass the select requests and responses to Lambda

## Snowball
Used to move large amounts of data in and out of AWS (mostly S3 and Glacier).  Replaced the Import/Export service). Convenient way to tranfer 100 TB to S3 in a week or less.
* Snowball
    * Petabyte scale suitcase for moving data
    * Simple, fast and secure. Scrubbed by AWS
    * 50 (42) TB or 80 (72) TB
* Snowball Edge
    * Adds compute capacity via Lambda functions
    * Storage and compute builtin
    * 100 (83) TB (45) clustered
    * 5 to 10 devices per cluster
    * Options for storage (24 vCPU), compute (54 vCPU) or GPU (54 vCPU) optimized
* Snowmobile
    * Trucksize Snowball
    * 100 PB

## Route 53
### 101
Domain Name System (DNS) translates IP addresses to human readable friendly names and vice versa
* IPv4 32-bit address space
* IPv6 64-bit address space
* Name servers have the DNS records for a domain
* Domain registrar 3rd party used to register domains
* Start of Authority (SOA) contains information about the DNS zone and associated records
* ELBs provide names and never IP addresses
* A records convert directly to an IP address
* CNAME records convert to another name which can then be converted to an IP address
* Time to Live (TTL) is the time the DNS record will be cached for. Reduce the TTL to propagate changes quickly
* Alias records can be used with naked domain names and CNAME records cannot e.g. m.example.com can have an ALIAS of example.com but not a CNAME
* Alias records are better / faster because AWS automatically reflects ELB IP address changes
* Alias records can point directly to AWS resources by name and automatically gets updated
* Uses health checks to determine if your application is healthy and use that in routing traffic

### Route53 Resolver
Used in hybrid environments to route DNS queries between your VPC and on premise networks
* Inbound and outbound
* Inbound only allows queries into your VPC based on an endpoint
* Outbound allows queries to your on premise network

### Policies
Traffic flow visual editor allows you to create different policies and version them
* Simple Routing (default)
    * Returns all the IP addresses in the policy in random order
    * Single or multiple IP addresses
* Weighted Routing
    * Returns the IP address response based on a weighted / fractional value specified
    * Multiple records
    * Blue / Green deployments to send fractional amounts or traffick
* Latency Routing
    * Records created per region
    * DNS query determines which regions have records and which region has the lowest latency for the requester
* Failover Routing
    * Used for Active / Passive setups
    * Uses health check to validate the primary / active node health
    * Returns the IP address of the passive node when the active node health check fails
* Geolocation Routing
    * Routed by location in the world you are coming from
* Geoproximity Routing
    * Can only be created by Traffic Flow editor
    * You can create proximities around each region and then set boundaries for routing to each region
* Multivalue Routing
    * Randomly returns multiple IP addresses up to 8 so if the first is not available the client software will try the rest

## Databases
### 101
* RDS (OLTP) - Relational DB Types - Tables, rows, fields
    * MySQL
    * MS SQL Server
    * Oracle
    * Aurora
    * MariaDB
    * PostgreSQL
* NoSQL/JSON
    * DynamoDB
* RedShift (Data Warehousing)
* Elasticache (In memory caching)
    * In memory caching used to deploy, operate and scale performance of web applications
        * Memcache
        * Redis

### RDS
* Automated backups
    * Enabled by default
    * Restore to any point (to a second) within retention period
    * Retention period 1-35 days
    * Daily snapshots & transaction logs
    * Backups free to the size of the database
    * Stored in S3
    * IO suspended during backup window
    * Customer sets backup window
    * Deleted when you delete the DB (take a user initiated backup i.e. a snapshot to preserve a database copy)
* Snapshots
    * User initiated
    * Stored until you delete it
* Restores
    * Creates a new database with new name
* Encryption at rest with KMS
    * To encrypt copy the snapshot with the encryption enabled then restore the snapshot to a new database
* Replication
    * Multi-AZ
        * Replicates the database synchronously to another AZ
        * Primary failures move the name to the secondary database in the next AZ automatically as a part of the failover process
        * Used for HA
        * Supported by Aurora, SQL Server, MySQL, PostgreSQL, MariaDB & Oracle
        * Supports read replicas
        * Single endpoint name (see automatic name move)
        * Automated backups are taken from the standby
        * Must be in the same region
        * Standby cannot be used for as the engine is inactive
        * Upgrades take place on the primary instance
    * Read replicas
        * Replicated asynchronously
        * Send read requests to read replicas to improve performance
        * Not for DR
        * Up to 5 replicas
        * Can be used with Multi-AZ
        * Automated backups must be enabled
        * Supported by MySQL, MariaDB, PostgreSQL & Aurora
        * Can also have read replicas but watch latency
        * Each replica has unique endpoint name
        * Can be prompted to master (must rebuild replication out manually)
        * Extendable to cross-AZ and cross-region

### DynamoDB
NoSQL database
* Name value pair must not exceed 400KB
* Millisecond access times
* SSD Storage
* Across 3 facilities in a region
* Global tables allows you to span regions
* Row = Item
* Eventually consistent model (default and best performance)
* Strongly consistent model available (use when read required 1 second after write)
* Expensive for large number of writes, cheap for large number of reads
* Read / Write capacity units is the important metric. Scales dynamically as you make the changes
* Query non-primary key attributes using secondary indices
* Scales to unlimited but above 10000 write / read capacity unites call AWS support

### RedShift
Online Analytical Processing (OLAP / Data Warehousing) to analyze massive amounts of data
* Single node (160GB)
* Multi node
    * Leader node - client connection (no charge)
    * Compute nodes (charge), up to 128
    * Data automatically distributed across nodes
* Columnar data storage reduces IO
    * Stores data together as columns rather than rows
    * Reduces IO since only the column is fetched rather than entire row
    * Allows for easy compression
* Charged for compute nodes, data transfer and backup
* Encrypted in transit via SSL
* Encrypted at rest with KMS or CloudHSM keys
* Loaded from S3, EMR, DynamoDB
* Handles own keys or via KMS
* Does not require indexes
* Operates in a single AZ
* Keeps 3 copies (original, replica and backup in S3)
* Uses Massively Parallel Processing (MPP) to distribute queries and loads
* Two node types
    * Dense compute (dc): high compute performance but less storage
    * Dense storage (ds): lots of storage

### Elasticache
* Great for reducing read stress on a database that is not changing frequently
* Memcache is a single AZ solution
* Redis can be multi-AZ

### Aurora
* MySQL & PostgreSQL compatible
* Commercial DB at minimal cost
* Scales to over 10TB
* Scales to 32 CPU / 244GB
* Keeps 2 copies in each AZ across 3 AZ
* Up to 15 replicas
* Can span multiple regions with Aurora Global Database
* Backups and failover are handled automatically

### Aurora Serverless
Automatically starts and scales up to application needs and is good for infrequent, intermittent and unpredictable workloads

## Simple Queuing Service (SQS)
Distributed queue system for message awaiting processing (always a pull based system)
* Great way to decouple applications components
* Messages can contain up to 256KB of data
* Messages can stay from 1 minute to 14 days (default is 4 days)
* Visibility timeout is the amount of time a message is hidden from the queue after being picked up for processing
* Default visibility timeout is 30 seconds with a maximum of 12 hours. Configure based on the time to process each message
* Messages will be processed at least once
* Short polling returns immediately even when no messages are in the queue
* Long polling checks periodically and only returns a response when a message exists or timeout is reached
* Queue types
    * Standard queue - best effort ordering
    * FIFO queue - guaranteed ordering
        * Limited to 300 transactions / second

## Simple Work Flow (SWF)
Coordinates work across distributed application components
* Workers (can be human) are programgs that interact with SWF to get tasks, process received tasks and return results
    * Tasks are assigned only once and never duplicated
* Deciders are programs that control the coordination of tasks and scheduling accoring to the application logic
* Workflow starters initiate the workflow
* Workflows are scoped in a domain for isolation from other tasks and executions within the same account
* Maximum workflow duration is 1 year measured in seconds
* Related workflows are grouped into domains which need to be registered
* Actors
    * Workflow starters - initiators
    * Deciders - flow control
    * Activity - task doer

## CloudFormation
Used to automate the provisioning of resources in AWS
* Written in JSON or YAML
* If error encountered during a deploy a ROLLBACK_IN_PROGRESS state will be assumed
* Templates larger than 51,200 bytes have to be loaded from S3 not direct upload
* NestedStacks allows you to break larger templates into smaller ones
* A template MUST have at least 1 resource
* Components of a CloudFormation Template
    * Metadata extra information about template
    * Description describe the template purpose
    * Parameters allows users to pass information into template
    * Transforms apply macros
    * Outputs are exported from the current stack for import into other stacks
    * Mappings map keys to values for use like a lookup table
    * Resources define what to provision
    * Conditions determine where a resources are created and with which attributes

## OpsWorks
* Orchestration service using Chef or Puppet
* Chef cookbooks & recipes
* Puppet manifests

## Simple Notification Service (SNS)
Setup operate and publish/push messages from the cloud to subscribers via topics or other applications
* Text (SMS), email, SQS, Lambda or HTTP endpoint
* Instant push no polling

## Elastic Transcoder
Converts media files from one format to another for playing on various platforms (PC, mobile phones, tablets, etc.)
* Intelligence built in to determine best format for target device
* Billed per minute and resolution of video output

## API Gateway
Managed service to publish, maintain, monitor and secure APIs at scale
* API caching improves performance by storing response for a specified TTL
* Web pages need the same origin policy so use cross-origin resource sharing (CORS) to address
* Throttle requests to prevent attacks
* Log results using CloudWatch Logs

## Kinesis
Streaming data has generated continuously from thousands of sources, sending small data records simultaneously (stock prices, social network data, etc.)
* Makes it easy to analyze and process streaming data
* Producers send data to streams which are then access to by consumers
* Great way to bring big data (news logs, social media feeds) into the AWS cloud for consumption by RedShift for business intelligence or Elastic Map Reduce for big data processing
* Course services
    * Streams.
        * 24 hour default to seven day retention
        * Uses shards
    * Firehose
        * No automatic data retention
        * Analysed and or sent to another device
        * Sends data to S3
        * No need to worry about consumers
        * No shards
    * Analytics
        * Run SQL queries of data in Kinesis.

## Elastic Container Service (ECS)
Build, test and deploy applications quickly using containers
* Highly reliable
* Infinitely scalable
* Standardized into containers
* Smaller than virtualization (no redundant software needed so higher density)
* Benefits
    * Consistent delivery
    * No worries about dependencies as they are bundled in the container
    * Better resource management
    * Very portable
    * Microservice
* Components
    * Docker image (readonly)
    * Docker container (launched from the image)
    * Layers / Union File System
    * Dockerfile
    * Docker engine
    * Docker client
    * Docker registry / Store / Hub
* Amazon EC2 Container Registry (ECR) is the AWS Docker Registry
* Task definitions
    * Required to run Docker containers
    * Task definitions contain instructions
    * Parameters for running CPU, memory, links to other containers
    * JSON format
    * Network Configuration
    * Startup commands for application
    * Environmental variables
* Auto scales if a task fails
* ECS Clusters
    * Default cluster provided per region
    * Contains different instance types
    * Only one cluster
    * IAM restricts access
* ECS Scheduler
    * Handles ELB registration
    * Container agent installed automatically on special ECS AMI (no windows)
    * EC2 instances use IAM roles to access ECS etc.
    * Access to EC2 for ECS is allowed

## Workspaces
AWS virtual desktop infrastructure integrated with active directory for authentication
* Accessible from Windows, PC, iPAD, etc.
* Windows 7
* Can be personalized
* Local administrator available
* Persistent
* D:\ automatically backed up every 12 hours
* No AWS account needed

## Active Directory Integration
* SAML authentication supported
* ADFS -> AD -> SAML -> AssumeRolewithSAML -> Teporary credentials assigned

## AWS Organizations (All features or consolidated billing)
Account management service that enables you to consolidate multiple AWS accounts to centrally manage them via policies
* Automate AWS account creation and management
* Control access to AWS Services (service control polices)
* Consolidate billing
    * Paying account manages linked accounts at the organization level broken out by accounts
    * Linked accounts are independent (20 limit)
    * Volume pricing
    * Easier to track charges
    * Reserved EC2 instances
        * Will use all available reserved instances in the linked accounts before going to on demand pricing
* Always multifactor root account with strong & complex password
* Paying account for billing only no resource deployment
* Enable monitoring on the paying account for billing alerts to include all linked account data
* Manage all features
* Organizations unites which you can apply policies to

## AWS Support
* Basic
* Developer
* Business
* Enterprise

## Cross Account Access
* Work productively in multiple AWS accounts
* Users groups, roles and policies to achieve
* Needs both account numbers

## Tagging
* Uses key value pairs
* Is metadata for resources
* Tags can be inherited
* Case sensitive
* Very important for pricing allocation

## Resouce Groups
* Makes it easy to group resources based on tags assigned e.g. Region, Name or Project
* Classic Resource Groups are global
* AWS Systems Manager Groups are per region and you can run commands against them

## Well Architected Framework
### Business Benefits
* Zero upfront infrastructure investment
* Just in time infrastructure
* More efficient resource utilization
* Usage based costing
* Reducted time to market

### Technical Benefits
* Scriptable infrastructure for automation aka. Infrastructure as Code
* Auto-scaling
* Proactive scaling
* Efficient development lifecycle
* Improved testability
* Disaster recovery and business continuity
* Overflow traffic to the cloud

### Design Principles
* Design for failure
* Decouple the components think SQS
* Elasticity (time-based, event-based or auto-scaling metric based)

### General Principles
* Stop guessing capacity
* Test systems at production scale (quick & cheap with cloud)
* Automate to make experimentation easier
* Allow for evolutionary architectures
* Data-driven architectures
* Simulate important days for testing

### 5 pillars of Architecture
* Operational Excellence
    * Automate change execution and responses
    * Plan changes and account for operational events
    * Perform operations with code
    * Align operations to business objectives
    * Make regular small incremental changes
    * Test for response to unexpected events
    * Learn from past events and failures
    * Keep standard operating procedures updated
    * Standardize and automate operations
    * Avoid downtime and large changes
    * Centralize and automate monitoring
* Cost optimization
    * Use to reduce operating costs to a minimum and use the cost savings for other business activity
    * Transparent assignment of cost for clear expenditure allocation
    * Use managed services to reduce cost of ownership
    * Trade capital expense for operating expense
    * Benefit from the economies of scaale (in short use the cloud)
    * Stop spending on data center operations (see economies of scale)
    * Definition
        * Match supply to demand (serverless, autoscaling)
        * Use cost effective resources
        * Expenditure awareness (provision in seconds, use tags to track costs, turn off what is not in use, budgets and billings alerts)
        * Optimizing over time (investigate new services and resources to see if it works better for you)
* Security
    * Apply security at all levels
    * Enable traceability
    * Automate responses to security events
    * Focus on securing your system (shared responsibility model)
    * Automate security best practices
    * Definition
        * Data protection
            * Versioning
            * Never moves the data to other regions without your permission
            * Encrypt at rest and in motion
            * Rotate keys
        * Privilege management
            * Role-based access
            * Access Control Lists
            * Password management policies
            * Multifactor Authentication is a must
        * Infrastructure protection
            * Secure the VPC
            * AWS handles the physical infrastructure protection
        * Detective controles
            * Use AWS Config, CloudTrail and CloudWatch
* Performance Efficiency
    * Using compute resources efficiently to meet requirements
        * Democratize advanced technologies
        * Go global in minutes
        * Use serverless architectures
        * Experiment more often
        * Definition
            * Compute
            * Storage
            * Database
            * Space-time tradeoff
* Reliability
    * System recovers from infrastructure failure or service disruption
        * Test recovery
        * Automatically recover from failure
        * Scale horizontally to increase availability
        * Stop guessing capacity
    * Definition
        * Foundations - think service limits
        * Change management - effect on application and infrastructure
        * Failure management - assume everything will fail
