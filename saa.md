# AWS Solution Architect - Associate
## AWS Global Infrastructure
* An Availability Zone (AZ) is one or more discrete data centers, each with redundant power, networking and connectivity housed in separate facilities
* A Region is a physical location in the workd which consists of two or more Availability Zones (AZ)
* Edge Locations are endpoints for AWS which are used for content caching. Typically consists of CloudFront, Amazon's Content Delivery Network (CDN)

## IAM
### 101
* Centralized control of AWS account
* Shared access to AWS account
* Granular permissions
* Identity Federation (Active Directory,  Facebook, LinkedIn, etc.)
* Multi-factor authentication (enable at least for root account)
* Provide temporary access for users / devices and services where necessary
* Default access is No Access
* Roles are Global
* You can now attach / detach roles to running EC2 instances

### Authorization and Authentication
* Users, Groups, Roles, Policy Documents (JSON)
* Universal so region independent
* Root account has complete access
* New users have no access
* Programmatic access via Access Key ID and Secret Access Key. Shared only at creation so save them for SDK, CLI and API use
* Console access via user and password
* Multi-factor and password policy can be enabled

## EC2
Amazon Elastic Compute Cloud reduces the time required to obtain and boot new server instances. Scale up and down quickly. Pay only for what you need.
* Termination protection is off by default
* Root EBS is removed automatically by default on termination
* Cannot encrypt customer volumes on AWS AMI must use a custom and encrypt on copy
* Pricing Options
    * On Demand - pay a fixed rate by the hour no commitment
        * Good for low cost no upfront payment
        * Shortterm, spiky or unpredictable workloads that cannot be interrupted
        * Applications still in testing
    * Reserved - provides a capacity reservation with a big discount over a period of typically 1 or 3 years
        * Predictable and steady usage applications
        * Can be converted to different instance types
    * Spot - enables you to bid whatever price with even greater savings if your needs are flexible. Price is driven by market
        * Require applications with flexible start and end time
        * Must be interruptible
        * Lowest cost
        * Great for large capacity
        * Terminated when the market price goes above your bid price
        * When Amazon terminates then you are not charged for the partial hour
        * You terminate you are charged for the partial hour
    * Dedicated - Your own hosts but comes at a premium price
        * Goodfor regulatory that does not support multi-tenant virtualization
* EBS Backed
    * Persistent
    * Attach and reattach (root requires outage)
    * Data remains if volumes are stopped
    * Default is to terminate if volumes are stopped
    * You need a custom AMI to encrypt the root volume
    * Volume must be in the same AZ as the EC2 instance
    * To encrypt a root device you can take a snapshot and then copy it and select encrypt then create an AMI from it to boot EC2 instances in different regions
    * Stop instance before taking a root volume snapshots to ensure consistency
    * Cannot share encrypt snapshots
    Volumes restored from encrypted snapshots are also encrypted
* Instance Backed
    * Not persistent (ephemeral)
    * Not for long term data
    * Data is deleted when terminated (no stop option)
    * Cannot attach EBS volumes during launch only after launch
    * Cannot be stopped
    * If the virtualization host fails you VM and instance store data are lost
    Launched from templates stored in S3 so a little slower to launch
* Metadata
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
## Auto Scaling
* Launch configuration speicifies the AMI, Instance size, IAM role, bootstrap script, etc. Answers the question of what each EC2 instance launched should look like
* Autoscaling group specifies how many instances are launched, which network they are launched in and in which subnet (it applies this intelligently). Specifically a load balancer, health check and grace period. Also apply scaling policies beyond the initial settings for growning and shrinking. Notifications can be sent via SNS for launch, terminate and associated failures

## EBS
Elastic Block Storage that is attached to EC2 instances
* Size can be changed on the fly
* Use RAID 0 to virtually increase IOPS limits
* Must be in the same AZ as the EC2 instance (latency issue)
* Move AZ/region using snapshot and copy
* Volume Types
    * General Purpose (gp2)
        * Bootable
        * Up to 10000 IOPS
    * Provisioned IOPS (io1)
        * Bootable
        * I/O intensive applications e.g. NoSQL
        * Greater than 10000 IOPS
        * Up to 20000 IOPS per volume
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
    * Cannot share encrypted volumes
    * Application consistent snapshot options for RAID
        * Freeze the file system
        * Unmount the volume
        * Shutdown the EC2 instance
## CloudWatch
* Basic / Standard monitoring (5 minute interval / free tier)
* Detailed monitoring ( 1 minute / paid service)
* Can create dashboards and alarms (sent via SNS) which requires confirmation
* EC2 metrics
    * CPU
    * Disk
    * Network
    * Status checks
* CloudWatch events enable you to respond to state changes in AWS resources
    * You can use lambda functions to respond to the state changes
* CloudWatch Logs used to aggregate, monitor and store logs
    * Requires an agent installed to send logs
* Use scripts or CloudWatch Agent to send custom metrics

## Security Groups
* Up to 5 applied to each EC2 instance
* All outbound allowed by default
* All inbound blocked by default
* Stateful
* Changes are immediately effective
* There is no deny just allow

## Elastic Load Balancers (ELB)
Load Balancers spread load across resources to provide services
* Application Load Balancer (ALB)
    * Layer 7 (application aware)
    * Best suited for HTTP & HTTPS
* Network Load Balancer (NLB)
    * Best for TCP traffic where extreme performance is required
    * Layer 4
* Classic Load Balancer (being deprecated)
    * Sticky sessions
    * HTTP & HTTPS traffic
    * Not recommended for new deployments
    * 504 Gateway Timeout - means it is having a hard time reaching the down stream resource e.g. EC2 web application
    * X-Forward-For header allows you to see the public IP address of the person making the request
    * Use the DNS name as there is no public IP address released by AWS
    * Layer 4 & 7 but not as advanced as the ALB

## Elastic File System (EFS)
* Storage capacity shrinks and grows as needed
* Available over NFSv4
* Can be shared across multiple EC2 instances
* Scale to petabytes
* Block based storage
* Read after write consistency
* EC2 must have the same security group as the EFS to mount the shared volume

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
## VPC
Logical datacenter in AWS
* 1 subnet = 1 availability zone
* Security groups are stateful
* Security groups do not span VPCs

* One internet gateway per VPC
* Configure route tables between subnets
* Launch EC2 into subnet of choice
    * New NACL default is DENY everything
    * Default NACL allows everything
    * Can use one NACL for multiple subnets
    * Rules are evaluted in numerical order (lowest first)
    * Separate inbound and outbound rules
    * Network Access Control Lists (NACLs) are stateless (must open ports directly, remember ephemeral ports)
    * Allow ephemeral ports for outbound
    Useful for blocking inbound traffic
* Default VPC
    * All subnets are Internet Accessible
    * EC2 instances have public and private address
* VPC flow logs allow you to capture the traffic sent within the VPC. It can be done at VPC, subnet or NIC level
    * Cannot tag flow log
    * Cannot enable flow logs on peer VPC unless it is in your account
    * Cannot change the configuration of a flow log after creation
    * Amazon core infrastructure (DNS, DHCP, etc.) is not included
* VPC Peering
    * Route traffic between VPC in same or different accounts
    * No single point of failure or bandwidth limitations
    * Cannot use the same or overlapping IP address (CIDR) block
    * No transitive peering. Must be directly peered
    * Cannot route direct connect of internet traffic. No edge routing
* Nat Instances
    * Replaced by more redundant NAT Gateway
    * Must disable source and destination traffic checking so that it can forward traffic from other instances
    * Must be in a public subnet
    * Instance size if determined by traffic processed
    * Must have a path from private subnet to NAT instance to get out
* Bastion Hosts / Jump Boxes
Used to securely administer EC2 instances in private subnets via SSH / RDP
    * Must be hardened
    Do not storage private keys use SSH Forwarding

## S3
### 101
Key value based object store
* S3 is object-based storage allows you to upload and download files
* Files can be 0bytes - 5TB
* There is unlimited storage available
* Files are stored in buckets
* S3 bucket names are universal. That is they must be unique globally
* Link to file example https://s3-eu-west-1.amazonaws.com/acloudguru
### Buckets
* Universal namespace
* Upload an object to S3 receive HTTP 200 code
* Types
    * S3 - Standard
        * Availability 99.9%
        * Durability 99.999999999% (11 9's)
    * S3 - Infrequent Access
        * Availability 99.9%
        * Durability 99.999999999% (11 9's)
        * Paid retrieval but quick
        * 30 days old & minimum size charged 128KB to transfer from standard
    * S3 - Reduced Redundancy Storage (S3 - One Zone Infrequent Access)
        * Availability 99%
        * Durability 99.999999999% (11 9's)
        * Single AZ
* Control access to buckets with either bucket ACL or bucket policy
* By default buckets are privateand all objects stored inside them are private
* Delete markers are replicated
* Deleting individual versions or delete markers will not be replicated
* You cannot replicate to multiple buckets or use a daisy chain
### Glacier
User to archive data with paid retrieval
* 30 days after IA or 1 day after S3
* Retrieval types
    * Expedited
    * Standard
    * Bulk
### Cross Region Replication
* Versioning must be enabled on source and target
* Regions must be unique
* Files existing in the bucket are not replicated automatically just new or updated files
### Versioning
* Storaes all version of an object (including all writes even if you delete the object)
* Great for backups
* Once enabled, versioning cannot be disable, only suspended
* Integrates with lifecycle rules
* Versioning has multi-factor integration to confirm deletes, providing additional security
### Lifecycle Management
* Can be used in conjunction with versioning
* Can be applied to current and previous versions
* Following actions can be done immediately
    * Transition to Standard - Infrequent Access storage class after 30 days
    * Transition to Glacier storage class 30 days after Infrequent Access if relevant
    * Permanently delete
### Transfer Acceleration
* Uses CloudFront Edge locations to speed up file transfers with S3
* Uses new URL e.g. bucketname.s3.accelerate.amazonaws.com
### Static Websites
* Policy to make the entire bucket public
* Websites that require database access cannot be hosted with S3 (unless you make calls to the API Gateway and Lamda)
* Scales automatically
* Sample URL bucketname.s3-websie-us-east-1.amazonaws.com (for website enabled)
### Security and Encryption
* New buckets are private
* Send access logs to separate S3 bucket
* Bucket policies are bucket wide vs. ACL which can go to the object level
* Encryption
    * Client side encryption
    * Server side encryption
        * Amazon S3 Managed Keys (SSE-S3)
        * KMS (SSE-KMS)
        * Customer Provided Keys (SSE-C)
    * In Transit
        * SSL/TLS

## CloudFront Content Delivery Network
* Edge location is the location where the content is cached. Separate from regions and AZ
* Origin is all the files that the CDN will distribute. Can be S3 bucket, EC2 instances, ELB instance, Route53 or non aWS file store
* Distribution is the name given to the CDN which consists of a collection of Edge locations
* Edge locations can be written to (process uploads)
* Objects are cached for a TTL
* Cache can be cleared but there is a charge
* Origin can be a folder in an S3 bucket
* S3 buckets do not have to be public and can limit access to just the distribution with OAI
* All HTTP methods are supported via the distribution including PUT and POST
* Restrict view access via using signed URLs and signed cookies
* Can further protect using WAF (Web Application Firewall)
* CNAMES can be specified
* Use OAI to access S3 origin and prevent everything else from accessing S3
* SSL can be a custom certificate
* Content can be restricted by Geography
* Types of CDN
    * RTMP / RTP for video streaming
    * Static website hosting

## Snowball
Used to move large amounts of data in and out of AWS (mostly S3).  Replaced the Import/Export service that had customers send in their own disks (chaos of types and connections)
* Snowball
    * Petabyte scale suitcase for moving data
    * Simple, fast and secure. Scrubbed by AWS
* Snowball Edge
    * Adds compute capacity via Lambda functions
    * Storage and compute builtin
* Snowmobile
    * Trucksize Snowball

## Route 53
### 101
Translates IP addresses to human readable friendly names and vice versa
* ELBs provide names and never IP addresses
* Alias records can be used with naked domain names and CNAME records cannot e.g. m.example.com can have an ALIAS of example.com but not a CNAME
* ALIAS records are better / faster because AWS automatically reflects ELB IP address changes
### Policies
* Simple Routing
    * Returns all the IP addresses in the policy in random order
* Weighted Routing
    * Returns the IP address response based on a weighted / fractional value specified
    * Multiple records
* Latency Routing
    * Records created per region
    * DNS query determines which regions have records and which region has the lowest latency to the region
* Failover Routing
    * Used for Active / Passive setups
    * Uses health check to validate the primary / active node health
    * Returns the IP address of the passive node when the active node health check fails
* Geolocation Routing
    * Routed by location in the world you are coming from
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
    * Multi-AZ replicates the database synchronously to another AZ
    * Primary failures move the name to the secondary database in the next AZ automatically as a part of the failover process
        * Used for HA
        * Supported by Aurora, SQL Server, MySQL, PostgreSQL, MariaDB & Oracle
        * Supports read replicas
        * Single endpoint name (see automatic name move)
    * Read replicas
        * Replicated synchronously
        * Send read requests to read replicas to improve performance
        * Not for DR
        * Up to 5 replicas
        * Automated backups must be enabled
        * Supported by MySQL, MariaDB, PostgreSQL & Aurora
        * Can also have read replicas but watch latency
        * Each replica has unique endpoint name
        * Can be prompted to master (must rebuild replication out manually)
### DynamoDB
NoSQL database
* Name value pair must not exceed 400KB
* Millisecond access times
* SSD Storage
* 3 disctince facilities in each region
* Eventually consistent model (default and best performance)
* Strongly consistent model available (use when read required 1 second after write)
* Expensive for large number of writes, cheap for large number of reads
* Read / Write capacity units is the important metric. Scales dynamically as you make the changes
* Query non-primary key attributes using secondary indices
* Scales to unlimited but above 10000 write / read capacity unites call AWS support
### RedShift
Online Analytical Processing (OLAP / Data Warehousing)
* Single node
* Multi node
    * Leader node - client connection
    * Compute nodes
* Columnar data storage reduces IO
* Charged for compute nodes, data transfer and backup
* Encrypted in transit via SSL
* Handles own keys or via KMS
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