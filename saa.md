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
