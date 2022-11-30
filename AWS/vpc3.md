# Hands-on VPC peering and VPC Endpoint

## Part 1 - Creating VPC peering between two VPCs ---> Default Vpc ----> myVPC

### STEP 1 : Launching Instances

- Launch two Instances. First instance will be in "az1a-private-subnet" of "myVPC",and the other one will be in your "Default VPC". 

- In addition, since the private EC2 needs internet connectivity to set user data, we also need NAT Gateway.

#### A. Configure Public Windows instance in **Default VPC.

```text
AMI             : Microsoft Windows Server 2019 Base
Instance Type   : t2.micro
Network         : Default VPC
Subnet          : Default Public Subnet
Security Group  : 
    Sec.Group Name : WindowsSecGrb
    Rules          : RDP --- > 3389 ---> Anywhere
Tag             :
    Key         : Name
    Value       : Windows public

- For MAC, "Microsoft Remote Desktop" program should be installed on the computer.
```
#### B. Since we need http connection we need to change Private Sec.Grb.

Security Group    : 
    Sec.Group Name : Private Sec.group
    Rules          : TCP  ---> 22 ---> Anywhere
                     HTTP ---> 80 ---> Anywhere

#### C. Since the private EC2 needs internet connectivity to set user data, we use NAT Gateway

- Click Create Nat Gateway button in left hand pane on VPC menu

- click Create NAT Gateway.

```bash
Name                      : my-nat-gateway

Subnet                    : az1a-public-subnet

Elastic IP allocation ID  : myElasticIP
```
- click "Create Nat Gateway" button

#### D. Modify Route Table of Private Instance's Subnet

- Go to VPC console on left hand menu and select Route Table tab

- Select "private-rt" ---> Routes ----> Edit Rule ---> Add Route
```
Destination     : 0.0.0.0/0
Target ----> Nat Gateway ----> my-nat-gateway
```
- click save routes

WARNING!!! ---> Be sure that NAT Gateway is in active status. Since the private EC2 needs internet connectivity to set user data, NAT Gateway must be ready.

#### E. Configure Private instance in 'az1a-private-subnet' of 'myVPC'.

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : myVPC 
Subnet          : az1a-private-subnet
user data       : 

#! /bin/bash
yum update -y
amazon-linux-extras install nginx1.12
yum install git -y
systemctl start nginx
cd /usr/share/nginx/html
git clone https://github.com/ethantechtorial/coffee-shop.git
chmod -R 777 /usr/share/nginx/html
rm index.html
cp -R ./designer/. .
systemctl restart nginx
systemctl enable nginx

Security Group    : 
    Sec.Group Name : Private Sec.group
    
Tag             :
    Key         : Name
    Value       : Private WEB EC2 
```

- Go to instance named 'Windows public' and push the connect button ----> Download Remote Desktop File

- Decrypt your ".pem key" using "Get Password" button
  - Push "Get Password" button
  - Select your pem key using "Choose File" button ----> Push "Decrypt Password" button
  - copy your Password and paste it "Windows Remote Desktop" program as a "administrator password"

- Open the internet explorer of windows machine and paste the private IP of EC2 named 'Private EC2 for peering'

- It is not able to connect to the website 


### STEP 2: Setting up Peering


- Go to 'Peering connections' menu on the left hand side pane of VPC

- Push "Create Peering Connection" button

```text
Peering connection name tag : First Peering
VPC(Requester)              : Default VPC
Account                     : My Account
Region                      : This Region (us-east-1)
VPC (Accepter)              : myVPC
```
- Hit "Create peering connection" button

- Select 'First Peering' ----> Action ---> Accept Request ----> Accept Request

- Go to route Tables and select default VPC's route table ----> Routes ----> Edit routes
```
Destination: paste "myVPC" CIDR blok
Target ---> peering connection ---> select 'First Peering' ---> Save routes
```

- select private-rt's route table ----> Routes ----> Edit routes
```
Destination: paste "default VPC" CIDR blok
Target ---> peering connection ---> select 'First Peering' ---> Save routes
```

- Go to windows EC2 named 'Windows public', write IP address on browser and show them to website.


WARNING!!! ---> Please do not terminate "NAT Gateway" and "Private WEB EC2" for next part.


## Part 2 - Create VPC Endpoint

### STEP 1: Configure S3 Bucket and Public Instances

Security Group    : 
    Sec.Group Name : Public-Sec-Group (Bastion Host)
    Rules          : TCP --- > 22 ---> Anywhere
                     All ICMP IPv4  ---> Anywhere
                     HTTP--------> Anywhere

Security Group    : 
    Sec.Group Name : Private-Sec-Group
    Rules          : TCP  ---> 22 ---> Public-Sec-Group
                     All ICMP IPv4 ---> Public-Sec-Group



#### A. Create S3 Bucket 

- Go to the S3 service on AWS console
- Create a bucket of `myvpc-<name>` with following properties, 

```text
Versioning                  : Disabled
Server access logging       : Disabled
Tagging                     : 0 Tags
Object-level logging        : Disabled
Default encryption          : None
CloudWatch request metrics  : Disabled
Object lock                 : Disabled
Block all public access     : unchecked
```
- upload 'image' files into the S3 bucket

#### B. Configure Public Instance (Bastion Host)

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : myVPC
Subnet          : az1a-public-subnet
Security Group  : 
    Sec.Group Name : Public Sec.group(Bastion Host)
    Rules          : TCP --- > 22 ---> Anywhere
                     All ICMP IPv4  ---> Anywhere
                     HTTP--------> Anywhere
Tag             :
    Key         : Name
    Value       : Public EC2 (Bastion Host)
```

#### C. Create IAM role to reach S3 from "Private WEB EC2"

- Go to IAM Service from AWS console and select roles on left hand pane

- click create role
```
use case : EC2 ---> Next : Permission
Policy ---> "AmazonS3FullAccess" ---> Next
Role Name : myS3FullAccessforEndpoint
Role description: my S3 Full Access for Endpoint
click create button
```
Go to EC2 service from AWS console

Select "Private WEB EC2" ---> Actions ---> Security ---> Modify IAM Role  select newly created IAM role named 'myS3FullAccessforEndpoint' ---> Apply

### STEP 2: Connect S3 Bucket from Private WEB Instance

#### A. Connect to the Bastion host

- Go to terminal and connect to the Bastion host named 'Public EC2 (Bastion Host)'

- Using Bastion host connect to the EC2 instance in "private subnet" named 'Private WEB EC2 ' (using ssh agent or copying directly pem key into the EC2)

- Enable ssh-agent (start the ssh-agent in the background)

```bash
eval "$(ssh-agent)"
```
- Add your private key to the ssh agent on your `localhost`. `ssh-agent is a program that runs in background and stores your keys in memory`.

```bash
ssh-add ./[your pem file name]
```
- connect to the ec2-in-az1a-public-sn instance in public subnet
```bash
ssh -A ec2-user@ec2-3-88-199-43.compute-1.amazonaws.com
```
#### B.Connect to the Private Instance

- once logged into the bastion host connect to the private instance in the private subnet:
```bash
ssh ec2-user@[Your private EC2 private IP]
```
#### C. Use CLI to verify connectivity

- list the bucket in S3 and content of S3 bucket named "aws s3 ls "myvpc-endpoint" via following command

```
aws s3 ls
aws s3 ls myvpc-endpoint
```
- go to private route table named "private-rt" on VPC service

- select routes sub-menu ---> Edit routes ---> Delete "Peering and NAT Gateway"

- Go to the terminal and try to connect again S3 bucket via following command
```
aws s3 ls
```
- show that you are "not able to connect" to the s3 buckets list


### STEP 3: Create Endpoint

#### A. Connect  to S3 via Endpoint

- go to the Endpoints menu on left hand pane in VPC

- click Create Endpoint
```text
Service Category : AWS services
Service Name     : com.amazonaws.us-east-1.s3
Service Type     : gateway
VPC              : myVPC
Route Table      : choose private one or both  
```
- Create Endpoint

- Go to private route table named 'private-rt' and show the endpoint rule that is automatically created by AWS 

#### B. Connect  to S3 via Endpoint

- Go to terminal, list the buckets in S3 and content of S3 bucket named "aws s3 ls "myvpc-<name>" via following command

```bash
aws s3 ls
aws s3 ls myvpc-<name>
touch connection-ok.txt
aws s3 cp connection-ok.txt s3://myvpc-<name>/
```













