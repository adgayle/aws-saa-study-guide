# AWS Solution Architect - Associate
## AWS Global Infrastructure
* Ab Availability Zone (AZ) is one or more discrete data centers, each with redundant power, networking and connectivity housed in separate facilities
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