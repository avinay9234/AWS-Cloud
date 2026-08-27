# AWS-Cloud
Class 1: IAM: Identity Access Management :
------------------------------------------
create user = username and go with default and create it

ARN: Any u want use resources like s3,EC2 u want grant u ARN.

IAM->users->click particular user->security credential enable console access ->enable->custom password->apply

->open incognito mode and login with IAM user

-> u can see example s3 resources to see any buckets is there its show permission denied

->IAM->policies->create new policies, click JSON 
{
"version": "2021-10-17",
"statement": [
{
"sid":"alloS3acess",
"effect":"Allow",
Action:"s3:*",
"Resource":"*",
}
]
}

after click next policy name :test-user-allow-s3-acess -> create policy

assign policy to that user
IAM->users->user name click ->add permission->attach policy directly,test-user-allow-s3-acess -> next and all policy

->again go to incognito just refresh it u can see error message was gone

Root Account -> Demo-group -> inside group 1.test-user ->create a policy

IAM->user group -> create group -> name of group: demo group ->create group

click demo group -> users -> add users ->select users and add it.

click demo group -> permissions -> attach policies -> select policy add it



class2:AWS Organization: 
------------------------
AWS Organizations -> AWS accounts -> check root ->action: create new ->Org unit name:test-OU -> create

check test-OU ->Ass an AWS Account ->AWS Account name: test-account, email: temporary i will give ->create AWS

it can create out in test-account just move to that test-OU
check test-account ->action, move and just select test-OU -> move AWS Account

->after u login another root account

class3:
--------
AWS Role : Assume Role
-----------------------
IAM Role:
---------
An IAM Role is just a set of permissions that can be temporarily assumed.

No username/password
No permanent credentials
Temporary access via STS (Security Token Service)

work flow:
----------
user -> IAM Role(s3 full access role) +IAM Policy(AmazonS3FullAcess)
In Amazon console:
------------------
Go To IAM -> Create User ->permission option: add user to group -> create user
->click on particular user -> security console ->enable console access ->apply
->roles-> select aws account ->An AWS account: This account ->next -> select:awss3full access ->next ->role name:S3-full-access ->create role
->Again go to users ->clock on ok particular user ->permission,add permission:create inline policy -> modify Action:"sts:AssumeRole" and Resource:"paste arn in roles" ->policy name:s3-full-accesss->create policy
->finally copy the role link and check it

Class:4
-------
What is Ec2 Instance
how to launch an Ec2 
how to create SHH key(public, private)
how to SSH into Ec2 instance

key pair: once create a key pair it create key two pair to generate pubic and private key private key is used to developer and public key used for EC2 instance

Allow SSH traffic :It allow means u can do the remote SSH or remotely login particular instance and up and running

class 5:
--------
->sign AWS cloud account
->create VPC(virtual private cloud) consider it is a data center inside VPC we can create subnets, internet gateway, route table
subnet can create two subnet Public subnet and private subnet

Public subnet: resources can exposes in the internet that can store to public subnet eg: web application ui part can store here 

private subnet : resources can not access into the internet that can store to private subnet eg:database
vpc ip range:12.0.1.0/16, public subnet:12.0.1.0/24, public subnet:12.0.2.0/24
->internet gateway: It is required to allow instances in a public subnet to send and receive traffic from the internet. Without an Internet Gateway, even if an instance has a public IP, it cannot communicate with the internet.
->Route Table: A route table is used to control traffic in a VPC. It contains rules that decide where the traffic should go, like sending internet traffic to an Internet Gateway. Based on the route table, we decide whether a subnet is public or private.
Step1:creating vpc
Step2:create internet Gateway
Step3:create Public and Private Subnet
Step4:Create Route Tables in public and private route tables
Step5: create Ec2 instances

Class6:
-------
->Bastion Host its also vpc setup

Class7:AWS EC2:User Data Script
--------------------------------
✅ Correct EC2 User Data Script (Ubuntu)

Use this instead:
#!/bin/bash

# Update packages
apt-get update -y

# Install Apache
apt-get install -y apache2

# Start and enable Apache
systemctl start apache2
systemctl enable apache2

# Create a simple web page
echo "<h1>Server is up and running</h1>" > /var/www/html/index.html

->not only paste we can upload file for shell script file also.

Class8:AWS EC2:Launch template & Source Template
------------------------------------------------

->create a launch template -> launch template name:ec2-instance ->template tag:name,ec2-instance ->AMI: Brows AMI select ubuntu -?instance type:t2.micro -> keypair -> Network settings: security groups:allow-ssh-traffic port and allow -traffic-port80 -> user data -> create launch template

->Go to launch instance drop down select launch instance from template -> choose launch template -> launch instance

VPC: security group : its for inbound out bound rules

Class9:AWS EC2:ALB(Application Load Balancer):
-------------------------------------
-> create VPC ip:12.0.0.0/16
-> create internet gateway 
-> create public two subnet with different availability zones ip:12.0.1.0/24, 12.0.3.0/24
-> create Route Table
-> We have to create two EC2 instances for two subnets. creating Ec2 instances in that time in network setting, add security group select http
-> create Target group :create TG ->instance, Target group name, vpc -> next -> select both instances: include as pending below -> create target group
->Load Balancer -> create LB -> select ALB -> LB name:, internet facing, VPC, mappings: select it, -> create one more security group for HTTP -> Listeners and routing: select target group -> create Load Balancer

class 10:Ec2 Auto Scaling :
--------------------------
-> create VPC ip:12.0.0.0/16
-> create internet Gateway
-> create public two subnet with different availability zones ip:12.0.1.0/24, 12.0.3.0/24
-> create Route Table
-> Create Target Group : No Instances
-> Create Load Balancer : here create a new security group for Http 
-> create Auto Scaling Group -> create AS -> name, create a new launch template [network setting: dont need subnet, select common security group,enable auto-assign public ip] -> next -> network :select VPC, select Available zones ->Load balancing: attach load balaner: chose from your LB target group:select target group ->Helth check : turn on check, Health check grace period:20sec ->desire:2, min:1, max:3  next ,next, next ->create Auto scaling


#!/bin/bash
sudo apt update -y
sudo apt install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2
echo "<h1>Server Details</h1>
<p><strong>Hostname:</strong> $(hostname)</p>
<p><strong>IP Address:</strong> $(hostname -I | awk '{print $1}')</p>" | sudo tee /var/www/html/index.html
sudo systemctl restart apache2


Class 11:How to use AWS Web application firewall/Web ACL:
---------------------------------------------------------
-> create VPC
-> create internet gateway
-> create public two subnet with different availability zones
-> create route table
-> create instance
-> create application load balancer: first create target group, now create Load balancer, in LB to create new security group
-> in search WAF & Shield -> click web ACLs -> select region location -> create web ACL -> name:AWS-waf-demo -> resource type: Regional resources, add AWS resources:select application load balancer select and add -> next -> before add rules in left side menu [ip sets ->ip set name: my-laptop-ip, ip addresses :laptop ip go goggle  serach  my lap ip ] add rules:add my own rules, rule type:ip set, name:block-my-laptop access, ip set:my-latop-ip, ip address to use the: source ip address, action:block ->add rule, select rule ->next,next,next -> create web ACL

class 12: VPC Peering
----------------------
-> create two VPC 12.0.0.0/16, 13.0.0.0/16
-> create two route table
-> create two subnet for vpc1 and vpc2 with same availability zone
-> create two internet gateway
-> launch instance for two vpcs

-> in vpc click on peering connections -> create peering connection, name:, select local vpc to peer with:vpc1,VPC ID:vpc2 -> create peering connection -> in Action: Accept request

->Route tables ->rt1->edit routes -> add route destination:destination of vpc2 ip, target:peering connection -> save changes
->Route tables ->rt2->edit routes -> add route destination:destination of vpc1 ip, target:peering connection -> save changes
-> after in putty connect to two instances after is peering or not u can check command like curl private ip address for another instance

class 13:AWS VPC Transits Gateway :
-----------------------------------
-> create VPC ip:12.0.0.0/16, 
-> create internet gateway
-> create subnet ip:12.0.1.0/24
-> create Route Table
-> create EC2 Instance
-> same way to create 2 vpc's we need three test-vpc
-> Transit Gateway -> create transit gateway, name: tg-vpc1-vpc2-vpc3 ,description: TG for vpc1 vpc2 vpc3, ASN:emty, -> create transit gateway
->Transit Gateway attachment ->name tag:tg-attachment-vpc-1->select transit gateway id -> attachment type: VPC -> VPC ID: select VPC-1 ->create transit gateway attachment
-> same way create remain transit gateways for two vpc's
->(vpc peering and transit gateway are same but in transit gateway you can create single transit gateway and handle multiple vpc's. In vpc peering more vpc are there you need to create multiple vpc peering)
-> update the route tables, route tables ->click on VPC1 route table id -> edit routes ->add routes -> destination: type second vpc2 ip, target: select transit gateway -> save changes
route table -> edit routes -> add routes -> destination: third vpc3 ip, target:transit gatway -> save changes

--- same way add routes on vpc2 and vpc3 ---

test:in terminal take 3 duplicates add permmsion and add ssh 
check i am able to access vpc2 and vpc 3 to check command curl vpc2private ip
finally we can saw the vpcs can communicate blw them



class 14:NAT Gateway setup in your VPC :
-------------------------------------
-> create VPC ip:12.0.0.0/16
-> create two subnet public and private same availability zones ip:ip:12.0.1.0/24, 12.0.2.0/24
-> create internet gateway
-> create two route table 
-> create NAT Gateway
-> After creating NAT gateway go to route table edit test-private route table add route destination:0.0.0/0 target: NAT Gateway -> save changes
-> create Two EC2 Instances Public and Private instances
-> Test NAT Gateway : in putty to access two instances private and public 


Class 15: AWS Route 53 course :
-------------------------------
->What is Route 53
->Hosted Zones
->Custom Domain + Goggle Domains
->Name server, A, CNAME, ALIAS
->Simple Routing, Weighted, Geolocation, Failover

1.Simple Routing:
-----------------
-> First u can create hosted zone -> Route 53 -> created hosted zone-> Domain name: test.xyz, Type:pubic hosted zone -> created hosted zone
->(Domain Name System) In DNS record changes, you need to copy the default name servers from the Route 53 hosted zone and update them in your domain registrar (like GoDaddy or Google Domains) by replacing the existing name servers.
-> After u can create Ec2 instance
-> I want to access my application using a domain name instead of an IP address. For that, I create an A record in DNS, which maps the domain name to the public IP address of an EC2 instance.
example.com → DNS → IP address → EC2 instance → Your app

->Route 53 -> Hosted zones -> click on domain name -> create a record -> routing policy: simple routing, ->next -> Define simple record, record name: subdomain emty, record type:A-Route traffic to an IPV4 address and some AWS resources, value/Route traffic to:paste ipv4 -> define simple record -> select and create record

2.Weighted Routing:
-------------------
-> create VPC ip:12.0.0.0/16
-> create Subnet two public(12.0.1.0/24,12.0.2.0/24) and one private subnet(12.0.3.0/24) with different availability zones
->create internet gateway and attach the VPC
->Create two Route Table for public and private
-> create a new Ec2 instance inside new VPC 
-> create target group: its is a logical grouping of Ec2 Instances so our load balancer can choose to this particular group to forword the request
-> create Application load balancer
->Route 53 -> Hosted zones -> click on domain name -> create a record -> routing policy: simple routing, ->next -> Define simple record, record name: subdomain emty, record type:A-Route traffic to an IPV4 address and some AWS resources, Value/Route traffic to: choose end point: Alias to Application and classic Load Balancer, choose region, choose load balancer - create records

More in Weighted Routing:
------------------------- 
Weighted: Weighted Routing is putting  percentage share the traffic load on the multiple load balancers

-> in this two VPC needed same setup for creating vpc2 13.0.0.0/16 all networking setup
-> create vpc2 EC2 instance
-> create target group
-> create Load Balancer
->Route 53 -> Hosted zones -> click on domain name -> create a record -> routing policy: Weighted routing ->next -> Define the Weighted records:choose endpoint:Application LB,availability zone,choose LB,weight:128[total 256 bits, 128 is 50%], Evaluate target health:yes, Record ID: A record for test-lb ->Define weighted route,and same way add one more A record for 2nd lb -> select 2 A record -> create records

3.Geolocation Routing
----------------------
-> All setup is same 

->Route 53 -> Hosted zones -> click on domain name -> create a record -> routing policy: Geolocation routing -> next, -> Define the Geolocation records ->choose end point,location,lb, loaction: sweden, Record ID: A record for Sweden test-lb , same way add one more -> create record
4.Failover Routing:
-------------------
-> All setup is same 
-> Route 53 -> Hosted zones -> click on domain name -> create a record -> routing policy: Failover -> next -> Define:Application LB,choose : primary,Record ID -> click on define -> same way define ->create record

--------------

Class 16:
----------
AWS certificate Manager:
-----------------------
->create VPC ip:12.0.0.0/16
->create two subnet ip:12.0.1.0/16,12.0.2.0/24
-> create internet gateway
->create Route table
->create EC2 instances
->create target group
->setup load balancer
-> setup the Route 53
->search AWS certificate Manager(ACM) ->Request Certificate ->next -> remaiming go with default ->request ->click certificate Id->Domains: create records in route 53 -> create record after u will see in Route 53 in hosted zones
->in load balancer add one more listener HTTP only is there but need to add HTTPS
-> in lb ->security -> click security group id -> add in bounce rule -> edit -> add rule ->select HTTPS 
->(u type http://demo.com but it will redirect to take https://demo.com) Got LB -> click on http:80 click on default, action: edit rule -> Defult action :redirect to url, protocal: HTTPS, port:443, (unselect custom host,path,query),status code:301-permanently moved -> save changes

Class 17 :AWS Lambda Function :
------------------------------
What is Lambda?
how to create your lambda
1.how to create lambda by uploading zip code file?
2.how to create lambda by uploading zip code file to s3 buckets?
3.lambda function url?
4.lambda environment variables?
5.lambda layers?

What is Lambda?
