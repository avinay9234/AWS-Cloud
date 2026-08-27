# AWS Cloud Learning Journey

This repository contains my hands-on AWS learning notes and practical implementations. It covers AWS Identity and Access Management, networking, compute, load balancing, auto scaling, security, DNS, certificates, serverless computing, and more.

## Technologies and AWS Services Covered

* AWS IAM
* AWS Organizations
* IAM Roles and AWS STS
* Amazon EC2
* SSH and Key Pairs
* Amazon VPC
* Subnets
* Internet Gateway
* Route Tables
* Bastion Host
* EC2 User Data
* Launch Templates
* Application Load Balancer (ALB)
* Target Groups
* EC2 Auto Scaling
* AWS WAF
* VPC Peering
* AWS Transit Gateway
* NAT Gateway
* Amazon Route 53
* AWS Certificate Manager (ACM)
* AWS Lambda

---

# Class 1: AWS IAM - Identity and Access Management

## What is IAM?

AWS Identity and Access Management (IAM) is used to securely manage access to AWS resources.

Using IAM, we can manage:

* Users
* Groups
* Roles
* Policies
* Permissions

## IAM User

An IAM user represents a person or application that needs access to AWS resources.

### Steps to Create an IAM User

1. Go to **AWS IAM**
2. Select **Users**
3. Click **Create user**
4. Enter the username
5. Create the user

To enable AWS Management Console access:

```text
IAM → Users → Select User → Security Credentials → Enable Console Access
```

Set a custom password and save the configuration.

You can then open an incognito/private browser window and log in using the IAM user credentials.

Initially, if the user tries to access an AWS resource such as an S3 bucket without permission, AWS returns an **Access Denied** error.

## ARN - Amazon Resource Name

An ARN uniquely identifies an AWS resource.

Example:

```text
arn:aws:s3:::my-bucket
```

ARNS are commonly used in IAM policies to specify which resources a permission applies to.

## IAM Policy Example - Full S3 Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Access",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

### Create and Attach the Policy

```text
IAM → Policies → Create Policy → JSON
```

Paste the policy and create it.

Example policy name:

```text
test-user-allow-s3-access
```

Attach the policy:

```text
IAM → Users → Select User → Add Permissions → Attach Policies Directly
```

After attaching the policy, refresh the IAM user's browser session. The user should now have the permissions defined in the policy.

## IAM Groups

IAM Groups allow permissions to be managed for multiple users.

Example:

```text
Demo Group
    ├── test-user-1
    ├── test-user-2
    └── test-user-3
```

### Steps

```text
IAM → User Groups → Create Group → Demo Group
```

Then:

```text
Select Group → Users → Add Users
```

Attach policies to the group:

```text
Select Group → Permissions → Attach Policies
```

Users inherit permissions attached to the group.

---

# Class 2: AWS Organizations

AWS Organizations helps centrally manage multiple AWS accounts.

It provides:

* Multi-account management
* Organizational Units (OUs)
* Centralized governance
* Billing management
* Policy management

Example structure:

```text
Root
│
├── Test-OU
│   ├── Test-Account-1
│   └── Test-Account-2
│
└── Production-OU
    └── Production-Account
```

## Steps

1. Open **AWS Organizations**
2. Create an Organizational Unit named `Test-OU`
3. Create an AWS account
4. Assign or move the account into the required OU

```text
AWS Organizations → AWS Accounts → Actions → Create AWS Account
```

---

# Class 3: IAM Roles and Assume Role

## What is an IAM Role?

An IAM Role is a set of permissions that can be temporarily assumed by AWS users, services, or accounts.

Important points:

* No username
* No permanent password
* No permanent access keys by default
* Temporary credentials are provided through AWS STS
* Permissions are controlled using IAM policies

## Example: S3 Full Access Role

Create a role:

```text
IAM → Roles → Create Role
```

Select:

```text
Trusted Entity Type: AWS Account
Account: This Account
```

Attach the required permission:

```text
AmazonS3FullAccess
```

Example role name:

```text
S3-Full-Access-Role
```

To allow an IAM user to assume the role, create a policy with `sts:AssumeRole`.

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "ROLE_ARN"
    }
  ]
}
```

Workflow:

```text
IAM User
    ↓
Permission to call sts:AssumeRole
    ↓
IAM Role
    ↓
Temporary Credentials from AWS STS
    ↓
Access AWS Resources
```

---

# Class 4: Amazon EC2

## What is an EC2 Instance?

Amazon EC2 (Elastic Compute Cloud) provides virtual servers in AWS.

EC2 can be used to:

* Host applications
* Run web servers
* Run backend services
* Perform development and testing
* Deploy workloads in the cloud

## Key Pair

An EC2 key pair consists of:

```text
Public Key  → Stored on the EC2 instance
Private Key → Downloaded and securely stored by the user
```

The private key is used to authenticate when connecting to the EC2 instance.

For Linux instances, SSH is commonly used:

```bash
ssh -i my-key.pem ubuntu@EC2_PUBLIC_IP
```

## Security Group

A Security Group acts as a virtual firewall for an EC2 instance.

Example inbound rules:

| Protocol | Port | Purpose             |
| -------- | ---: | ------------------- |
| SSH      |   22 | Remote Linux access |
| HTTP     |   80 | Web traffic         |
| HTTPS    |  443 | Secure web traffic  |

---

# Class 5: Amazon VPC

## What is a VPC?

Amazon Virtual Private Cloud (VPC) allows us to create a logically isolated network in AWS.

A VPC can contain:

* Subnets
* Route Tables
* Internet Gateway
* NAT Gateway
* EC2 Instances
* Security Groups

Example:

```text
VPC: 12.0.0.0/16

Public Subnet:  12.0.1.0/24
Private Subnet: 12.0.2.0/24
```

## Public Subnet

Resources can communicate with the internet when correctly configured with:

* Internet Gateway
* Route to Internet Gateway
* Public IPv4 address or equivalent public connectivity

Example:

```text
Web Server
Application Frontend
Load Balancer
Bastion Host
```

## Private Subnet

Resources are not directly accessible from the internet.

Example:

```text
Database
Internal Application Server
Backend Services
```

## Internet Gateway

An Internet Gateway enables communication between a VPC and the internet.

A public route table generally contains:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

## Route Table

A route table controls where network traffic is sent.

Example:

```text
Destination        Target
12.0.0.0/16        Local
0.0.0.0/0          Internet Gateway
```

## VPC Setup Steps

```text
Step 1 → Create VPC
Step 2 → Create and Attach Internet Gateway
Step 3 → Create Public and Private Subnets
Step 4 → Create and Configure Route Tables
Step 5 → Launch EC2 Instances
```

---

# Class 6: Bastion Host

A Bastion Host is an EC2 instance used to securely access instances in private subnets.

Architecture:

```text
Internet
    ↓
Bastion Host
(Public Subnet)
    ↓
SSH
    ↓
Private EC2 Instance
(Private Subnet)
```

The private EC2 instance does not need direct internet access for administrative SSH access through the bastion host.

---

# Class 7: EC2 User Data

EC2 User Data allows scripts to run automatically when an EC2 instance launches.

## Ubuntu Apache Installation Script

```bash
#!/bin/bash

apt-get update -y
apt-get install -y apache2

systemctl start apache2
systemctl enable apache2

echo "<h1>Server is up and running</h1>" > /var/www/html/index.html
```

User Data can be entered directly or provided through a script file, depending on the launch workflow.

---

# Class 8: EC2 Launch Templates

A Launch Template stores EC2 launch configuration.

It can include:

* AMI
* Instance type
* Key pair
* Security Groups
* Storage configuration
* User Data

Example:

```text
Launch Template Name: ec2-instance
AMI: Ubuntu
Instance Type: t2.micro
```

Security group rules:

```text
SSH: 22
HTTP: 80
```

Once created:

```text
EC2 → Launch Instance → Launch from Template
```

---

# Class 9: Application Load Balancer

An Application Load Balancer (ALB) distributes HTTP and HTTPS traffic across multiple targets.

Architecture:

```text
Users
  ↓
Application Load Balancer
  ↓
Target Group
  ↓
├── EC2 Instance 1
└── EC2 Instance 2
```

## Setup

1. Create a VPC
2. Create an Internet Gateway
3. Create two public subnets in different Availability Zones
4. Configure route tables
5. Launch two EC2 instances
6. Create a Target Group
7. Register EC2 instances as targets
8. Create an Application Load Balancer
9. Attach the Target Group to the ALB listener

Using multiple Availability Zones improves availability.

---

# Class 10: EC2 Auto Scaling

Auto Scaling automatically adjusts the number of EC2 instances based on demand and configuration.

Example:

```text
Minimum Capacity: 1
Desired Capacity: 2
Maximum Capacity: 3
```

Architecture:

```text
Users
  ↓
Application Load Balancer
  ↓
Auto Scaling Group
  ↓
├── EC2 Instance
├── EC2 Instance
└── Additional Instance when needed
```

## Setup

1. Create VPC networking
2. Create Target Group
3. Create Application Load Balancer
4. Create Launch Template
5. Create Auto Scaling Group
6. Select VPC and multiple Availability Zones
7. Attach the Target Group
8. Configure health checks
9. Set minimum, desired, and maximum capacity

## User Data Script

```bash
#!/bin/bash

apt update -y
apt install -y apache2

systemctl start apache2
systemctl enable apache2

echo "<h1>Server Details</h1>
<p><strong>Hostname:</strong> $(hostname)</p>
<p><strong>IP Address:</strong> $(hostname -I | awk '{print $1}')</p>" > /var/www/html/index.html

systemctl restart apache2
```

---

# Class 11: AWS WAF

AWS WAF (Web Application Firewall) helps protect web applications from unwanted HTTP and HTTPS requests.

It can be associated with services such as an Application Load Balancer.

Example architecture:

```text
User Request
     ↓
AWS WAF
     ↓
Application Load Balancer
     ↓
EC2 Instances
```

## Example: Block a Specific IP Address

1. Create an IP Set
2. Add the IP address or CIDR range
3. Create a Web ACL
4. Associate the Web ACL with an Application Load Balancer
5. Create a rule using the IP Set
6. Set the action to `Block`

Example:

```text
IP Set: my-laptop-ip
Rule: block-my-laptop-access
Action: Block
```

---

# Class 12: VPC Peering

VPC Peering allows private communication between two VPCs.

Example:

```text
VPC 1: 12.0.0.0/16
        ↕
   VPC Peering
        ↕
VPC 2: 13.0.0.0/16
```

## Important Step: Update Route Tables

VPC 1 Route Table:

```text
Destination: 13.0.0.0/16
Target: VPC Peering Connection
```

VPC 2 Route Table:

```text
Destination: 12.0.0.0/16
Target: VPC Peering Connection
```

After security groups and routes are correctly configured, instances can communicate using private IP addresses.

Example:

```bash
curl http://PRIVATE_IP
```

---

# Class 13: AWS Transit Gateway

AWS Transit Gateway provides a central networking hub for connecting multiple VPCs and networks.

Example:

```text
        VPC 1
          │
          │
    Transit Gateway
       /        \
      /          \
   VPC 2        VPC 3
```

This is more scalable than creating many individual VPC peering connections.

## Setup

1. Create multiple VPCs
2. Create a Transit Gateway
3. Create Transit Gateway Attachments for each VPC
4. Configure route tables in each VPC
5. Configure Transit Gateway routing as required
6. Test connectivity

Example:

```text
VPC 1: 12.0.0.0/16
VPC 2: 13.0.0.0/16
VPC 3: 14.0.0.0/16
```

Each VPC route table needs routes for the other VPC CIDR ranges through the Transit Gateway.

---

# Class 14: NAT Gateway

A NAT Gateway allows instances in a private subnet to initiate outbound internet connections while preventing unsolicited inbound internet connections.

Architecture:

```text
Private EC2
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

Example:

```text
VPC: 12.0.0.0/16

Public Subnet:  12.0.1.0/24
Private Subnet: 12.0.2.0/24
```

The NAT Gateway is placed in a public subnet.

Private route table:

```text
Destination: 0.0.0.0/0
Target: NAT Gateway
```

---

# Class 15: Amazon Route 53

Amazon Route 53 is AWS's DNS service.

It provides:

* Domain management
* Hosted Zones
* DNS records
* Health checks
* Traffic routing policies

Common record types:

```text
A
CNAME
Alias
```

## Simple Routing

Simple Routing directs traffic to a single resource.

Example:

```text
example.com
    ↓
DNS
    ↓
Public IP / AWS Resource
    ↓
EC2 Application
```

## Weighted Routing

Weighted Routing distributes traffic between multiple records based on assigned weights.

Example:

```text
Load Balancer 1 → Weight 128
Load Balancer 2 → Weight 128
```

This results in approximately:

```text
50% → Load Balancer 1
50% → Load Balancer 2
```

Important: The weights are relative values. They do not need to total 256.

## Geolocation Routing

Geolocation Routing sends traffic based on the user's geographic location.

Example:

```text
Users from Sweden → Load Balancer A
Users from India  → Load Balancer B
```

## Failover Routing

Failover Routing supports active-passive architecture.

```text
Primary Resource
       ↓
If Health Check Fails
       ↓
Secondary Resource
```

---

# Class 16: AWS Certificate Manager

AWS Certificate Manager (ACM) is used to provision and manage SSL/TLS certificates.

Architecture:

```text
User
 ↓ HTTPS
Application Load Balancer
 ↓
Target Group
 ↓
EC2 Instances
```

## Steps

1. Configure VPC and EC2 infrastructure
2. Create a Target Group
3. Create an Application Load Balancer
4. Configure Route 53 DNS
5. Request a certificate in ACM
6. Validate domain ownership
7. Add an HTTPS listener on port 443
8. Attach the ACM certificate to the listener
9. Allow HTTPS traffic in the ALB Security Group
10. Configure HTTP to HTTPS redirection

Example redirect:

```text
HTTP :80
   ↓
301 Permanent Redirect
   ↓
HTTPS :443
```

---

# Class 17: AWS Lambda

## What is AWS Lambda?

AWS Lambda is a serverless compute service that runs code without requiring you to manage servers.

With Lambda, AWS handles:

* Server provisioning
* Scaling
* Infrastructure management
* High availability

You focus mainly on writing and deploying your code.

## Lambda Topics Covered

### 1. Uploading Code as a ZIP File

You can package your Lambda function code and dependencies into a ZIP file and upload it.

### 2. Uploading Lambda Code from Amazon S3

For larger deployment packages, the ZIP file can be uploaded to Amazon S3 and then used during Lambda deployment.

### 3. Lambda Function URL

A Lambda Function URL provides a dedicated HTTPS endpoint for invoking a Lambda function.

Example:

```text
Client
   ↓
Lambda Function URL
   ↓
AWS Lambda
   ↓
Response
```

### 4. Lambda Environment Variables

Environment variables are used to provide configuration values without hardcoding them directly in the application code.

Example:

```text
DB_HOST=database.example.com
ENVIRONMENT=dev
API_URL=https://example.com
```

Sensitive values should generally be handled using appropriate services such as AWS Secrets Manager or Systems Manager Parameter Store rather than storing secrets insecurely.

### 5. Lambda Layers

Lambda Layers allow common dependencies or libraries to be packaged separately and reused by multiple Lambda functions.

Example:

```text
Lambda Layer
   ├── Python Libraries
   ├── Node.js Dependencies
   └── Common Utilities

        ↓ Used By

Lambda Function 1
Lambda Function 2
Lambda Function 3
```

---

# AWS Architecture Concepts Learned

During this learning journey, I practiced the following architectures:

```text
IAM User → IAM Policy → AWS Resources

Internet → ALB → Target Group → EC2 Instances

Internet → WAF → ALB → EC2 Instances

Internet → Bastion Host → Private EC2

Private EC2 → NAT Gateway → Internet Gateway → Internet

VPC 1 ↔ VPC Peering ↔ VPC 2

VPC 1
VPC 2 ─── Transit Gateway ─── Multiple Networks
VPC 3

Domain → Route 53 → Load Balancer → EC2

HTTP → ALB → HTTPS Redirect → ACM Certificate → Secure Application

Client → Lambda Function URL → AWS Lambda
```

---

# Learning Outcome

After completing these hands-on sessions, I gained practical understanding of:

* AWS Identity and Access Management
* AWS Account and Organization management
* IAM Roles and temporary access
* EC2 provisioning and SSH access
* VPC networking fundamentals
* Public and private subnet architecture
* Load balancing and high availability
* Auto Scaling
* Web Application Firewall configuration
* VPC-to-VPC connectivity
* NAT Gateway
* DNS and traffic routing using Route 53
* SSL/TLS certificate management using ACM
* Serverless computing using AWS Lambda

---

# Important Note

These notes are based on hands-on learning and practice. Some configurations such as CIDR ranges, instance types, and security rules should be adjusted based on real project requirements.

For production environments, always follow AWS security best practices, including:

* Use least-privilege IAM permissions
* Avoid using the root account for daily activities
* Protect and rotate credentials
* Restrict SSH access
* Avoid exposing private resources publicly
* Use Multi-AZ architecture where required
* Enable logging and monitoring
* Use appropriate backups and disaster recovery strategies
