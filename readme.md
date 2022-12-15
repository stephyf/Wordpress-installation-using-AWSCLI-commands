# Wordpress installation using AWSCLI commands
​
Hi! I'm here to discuss Wordpress installation using AWSCLI commands. I am planning to execute this setup using VPC and subnets which is very easy via AWS console. Now I am going to try this same setup using AWSCLI command. 
​
## Following are the resources we need to create for this setup:
​
- VPC 
- Subnets [2 Public subnet and 1 private subnet]
- Internet Gateway
- NAT Gateway
- Route tables [Public and private]
- Frontend-server 
- Backend-server
- Bastion-server
- Keypair
- Security groups
​
​
## Prerequisites
​
- Installing or updating AWSCLI
- Configuring the AWSCLI
​
### Installing or updating AWSCLI
​
As an intital setup, we need to install AWSCLI. Here I am going to install AWSCLI in one of the EC2 instance configured in the region ap-south-1. Basically, we don't want to install AWSCLI in EC2 instance as it has been already installed in it. 
​
The below command is used for installing awscli
````
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
````
You can update the awscli using the below command :
````
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
````
### Configuring AWSCLI
​
To do the same we need to create an IAM user with programmatic access. While creating user it will generate Access key and Secret key. Save the keys at your end safely and by using the below command we can configure AWSCLI. 
​
Command to configure AWSCLI
​
````sh
aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: ap-south-1
Default output format [None]: json
````
​
The AWSCLI stores this information in a profile (a collection of settings) named default in the credentials file. 
​
​
​
## Create a VPC and subnets
​
Here we are going to create VPC in the region us-east-2 with name "zomato" using the block 172.16.0.0/16. In this setup I am planning to do the wordpress installation with 3 subnets. 2 of them are public and one of them is private. For using these 3 subnets, I need to configure some other resources like internet-gateway, NAT-gateway, Route-tables. Let's see how we are using these resources in the coming procedures. Let's start the mentioned setup by creating VPC.
​
### Resources required in this step
​
- VPC
- 2 Public Subnets
- 1 Private Subnet
​
Below is the command to create VPC in the region us-east-2 with name "zomato" using the block 172.16.0.0/16.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-vpc --cidr-block 172.16.0.0/16 --region us-east-2
{
    "Vpc": {
        "CidrBlock": "172.16.0.0/16",
        "DhcpOptionsId": "dopt-043b33dac2dbec48c",
        "State": "pending",
        "VpcId": "vpc-024d0ce4d6547dcb6",
        "OwnerId": "526680641953",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-01afd04f10a7aeff7",
                "CidrBlock": "172.16.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false
    }
}
````
​
Tag That VPC with name "Zomato".
​
````
[ec2-user@ip-172-31-44-84 ~]$  aws ec2 create-tags --resources vpc-024d0ce4d6547dcb6 --tags Key=Name,Value=Zomato --region us-east-2
​
````
​
After creating VPC next, I am going to create 3 subnets. Two of them are public subnets and 1 of type private-subnet as we discussed above. And the 3 subnets will be created in the region us-east-2a, us-east-2b and us-east-2c respectively.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-subnet --vpc-id vpc-024d0ce4d6547dcb6 --cidr-block 172.16.0.0/18 --region us-east-2 --availability-zone us-east-2a
{
    "Subnet": {
        "AvailabilityZone": "us-east-2a",
        "AvailabilityZoneId": "use2-az1",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.16.0.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-07c95b8c2995dd6ef",
        "VpcId": "vpc-024d0ce4d6547dcb6",
        "OwnerId": "526680641953",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:us-east-2:526680641953:subnet/subnet-07c95b8c2995dd6ef",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
​
````
​
Tagged the first subnet with the name "zomato-public-1"
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-tags --resources subnet-07c95b8c2995dd6ef --tags Key=Name,Value=Zomato-public-1 --region us-east-2 --availability-zone us-east-2
​
````
Next, creating 2nd subnet with the name "zomato-public-2"
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-subnet --vpc-id vpc-024d0ce4d6547dcb6 --cidr-block 172.16.64.0/18 --region us-east-2 --availability-zone us-east-2b
{
    "Subnet": {
        "AvailabilityZone": "us-east-2b",
        "AvailabilityZoneId": "use2-az2",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.16.64.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-028a6d96fc96a387b",
        "VpcId": "vpc-024d0ce4d6547dcb6",
        "OwnerId": "526680641953",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:us-east-2:526680641953:subnet/subnet-028a6d96fc96a387b",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-tags --resources subnet-028a6d96fc96a387b --tags Key=Name,Value=Zomato-public-2 --region us-east-2 
​
````
​
After creating 2 public subnets now, I am going to create a private subnet with the name "zomato-private-1"
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-subnet --vpc-id vpc-024d0ce4d6547dcb6 --cidr-block 172.16.128.0/18 --region us-east-2 --availability-zone us-east-2c
{
    "Subnet": {
        "AvailabilityZone": "us-east-2c",
        "AvailabilityZoneId": "use2-az3",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.16.128.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0efb3c7d55f3662ac",
        "VpcId": "vpc-024d0ce4d6547dcb6",
        "OwnerId": "526680641953",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:us-east-2:526680641953:subnet/subnet-0efb3c7d55f3662ac",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-tags --resources subnet-0efb3c7d55f3662ac --tags Key=Name,Value=Zomato-private --region us-east-2
​
````
Thus we have successfully created VPC and 2 public subnets and private subnet.
​
## Make subnet public and private
​
To allow the subnets public, here I am going to create an internet gateway using the following create-internet-gateway command. And created a tag with the name "zomato-igw".
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-0dfeeb90f728b81f0
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-tags --resources igw-0dfeeb90f728b81f0 --tags Key=Name,Value=zomato-igw
````
​
Using the internet-gateway ID from the previous step, attach the internet gateway to VPC using the following attach-internet-gateway command.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 attach-internet-gateway --vpc-id vpc-024d0ce4d6547dcb6 --internet-gateway-id igw-0dfeeb90f728b81f0
​
````
There will be a default route table assigned to all subnets. Now I am going to create a custom route table for the VPC using the following create-route-table command and create a tag of "zomato-private-route".
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-route-table --vpc-id vpc-024d0ce4d6547dcb6 --query RouteTable.RouteTableId --output text
rtb-03119a1e230549f7e
​
````
​
Described the created route table which shows the details of the route-tables which we have created.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 describe-route-tables --route-table-id rtb-03119a1e230549f7e
{
    "RouteTables": [
        {
            "Associations": [],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-03119a1e230549f7e",
            "Routes": [
                {
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-024d0ce4d6547dcb6",
            "OwnerId": "526680641953"
        }
    ]
}
[ec2-user@ip-172-31-44-84 ~]$
````
?
Tagged the route table with name "zomato-private-route"
?
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-tags --resources rtb-03119a1e230549f7e --tags Key=Name,Value=zomato-private-route
​
````
​
The route table is currently not associated with any subnet. You need to associate it with a subnet in the VPC so that traffic from that subnet is routed to the internet gateway. Use the following describe-subnets command to get the subnet IDs. The --filter option restricts the subnets to your new VPC only, and the --query option returns only the subnet IDs and their CIDR blocks. Let's see the outputs:
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-024d0ce4d6547dcb6" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
[
    {
        "ID": "subnet-07c95b8c2995dd6ef",
        "CIDR": "172.16.0.0/18"
    },
    {
        "ID": "subnet-0efb3c7d55f3662ac",
        "CIDR": "172.16.128.0/18"
    },
    {
        "ID": "subnet-028a6d96fc96a387b",
        "CIDR": "172.16.64.0/18"
    }
]
​
````
?
Enable DNS hostname to VPC
?
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 modify-vpc-attribute --vpc-id vpc-024d0ce4d6547dcb6 --enable-dns-support
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 modify-vpc-attribute --vpc-id vpc-024d0ce4d6547dcb6 --enable-dns-hostnames 
````
​
Associate public route tables to public subnets and attach internet gateway
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 associate-route-table --route-table-id rtb-0ec07251dcc6bcb6b --subnet-id subnet-07c95b8c2995dd6ef
{
    "AssociationId": "rtbassoc-0ab096abf6c84e116",
    "AssociationState": {
        "State": "associated"
    }
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 associate-route-table --route-table-id rtb-0ec07251dcc6bcb6b --subnet-id subnet-028a6d96fc96a387b
{
    "AssociationId": "rtbassoc-03e092e20cc2066b8",
    "AssociationState": {
        "State": "associated"
    }
}
````
​
After associating, I am going to attach internet gateway to the default route table or the route table assigned to the public subnets.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-route --route-table-id rtb-0ec07251dcc6bcb6b --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0dfeeb90f728b81f0
{
    "Return": true
}
[ec2-user@ip-172-31-44-84 ~]$
````
?
Then I am going to create NAT Gateway and allocate to the second public subnet with the name "zomato-public-2". For creating NAT gateway I am going to create elastic ip address first using the below command:
?
````
ec2-user@ip-172-31-44-84 ~]$ aws ec2 allocate-address --domain vpc
{
    "PublicIp": "52.14.91.130",
    "AllocationId": "eipalloc-0e2e04c49a9bbf72a",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "us-east-2",
    "Domain": "vpc"
}
````
​
Then created NAT gateway and allocate the elastic ip which we created.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-nat-gateway --subnet-id subnet-028a6d96fc96a387b --allocation-id eipalloc-0e2e04c49a9bbf72a
{
    "ClientToken": "28ea68ce-c6b7-4707-b57e-fbd1e0e22171",
    "NatGateway": {
        "CreateTime": "2022-12-09T14:15:47+00:00",
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-0e2e04c49a9bbf72a"
            }
        ],
        "NatGatewayId": "nat-0f6dfa8491b9b9e92",
        "State": "pending",
        "SubnetId": "subnet-028a6d96fc96a387b",
        "VpcId": "vpc-024d0ce4d6547dcb6",
        "ConnectivityType": "public"
    }
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-tags --resources nat-0f6dfa8491b9b9e92 --tags Key=Name,Value=zomato-nat
​
````
​
Then I am going to associate private route table to private subnet and attach NAT gateway to this.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 associate-route-table --route-table-id rtb-03119a1e230549f7e --subnet-id subnet-0efb3c7d55f3662ac
{
    "AssociationId": "rtbassoc-0d14a5f2f0dbceade",
    "AssociationState": {
        "State": "associated"
    }
}
[ec2-user@ip-172-31-44-84 ~]$ 
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-route --route-table-id rtb-03119a1e230549f7e --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-0f6dfa8491b9b9e92
{
    "Return": true
}
 
````
Thus we have completed the VPC creation part successfully. For the same we have created subnets, route tables, internet gateway, NAT gateway and associated with the respective subnets.
​
## Launch instances into your subnets
​
For launching instance, we need to create Keypair, security groups for each instances. In our scenario, we are creating 3 instances in 3 subnets and for each instances we are creating 3 different security groups. 
​
### Resources required in this step
​
- Keypair
- 3 Security groups
- 3 Instances
​
First, we are going to create Keypair using the create-key-pair command:
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-key-pair --key-name MyKeyPair --query "KeyMaterial" --output text > MyKeyPair.pem
````
​
Then I am going to create 3 security groups. One security group only have SSH access and assigning to one of our public subnet. And the other one which I am going to create with only have httpd and SSH access from the first instance, because I am going to create with only httpd and php content installed in that instance. After doing the same, I am going to create another instance which going to have only database and it is going to setup in private subnet with allowing only SSH from our first instance who has SSH access and allowing port for Mysql access. Let 's see the security group creation for this scenario:
​
Here I am going to create with the 3 different names:
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-security-group --group-name zomato-bastion --description "MyIP" --vpc-id vpc-024d0ce4d6547dcb6
{
    "GroupId": "sg-072fbdd8f5c28ab66"
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-security-group --group-name zomato-frontend --description "Allow 80 and bastionip" --vpc-id vpc-024d0ce4d6547dcb6
{
    "GroupId": "sg-0537d3a2d2f41e2eb"
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 create-security-group --group-name zomato-backend --description "Allow 3306 and bastionip" --vpc-id vpc-024d0ce4d6547dcb6
{
    "GroupId": "sg-08808fb3017fe8ee8"
}
````
​
Then I am going to assign the rules in the respective security groups:
​
Rule added in the security group for the instance which is only using for SSH access:
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 authorize-security-group-ingress --group-id sg-072fbdd8f5c28ab66 --protocol tcp --port 22 --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0db642b79f598fd9c",
            "GroupId": "sg-072fbdd8f5c28ab66",
            "GroupOwnerId": "526680641953",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
````
​
Then I am going to add port 80 access and SSH from the instance which craeted only for SSH access:
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 authorize-security-group-ingress --group-id sg-0537d3a2d2f41e2eb --protocol tcp --port 80 --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00b7fc636c31fe32a",
            "GroupId": "sg-0537d3a2d2f41e2eb",
            "GroupOwnerId": "526680641953",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 authorize-security-group-ingress --group-id sg-0537d3a2d2f41e2eb --protocol tcp --port 22 --source-group sg-072fbdd8f5c28ab66
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0de89e93d7d9292dd",
            "GroupId": "sg-0537d3a2d2f41e2eb",
            "GroupOwnerId": "526680641953",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "ReferencedGroupInfo": {
                "GroupId": "sg-072fbdd8f5c28ab66",
                "UserId": "526680641953"
            }
        }
    ]
}
````
Then I have created another security group allowing 3306 port access and SSH from the instance which created only for SSH access:
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 authorize-security-group-ingress --group-id  sg-08808fb3017fe8ee8 --protocol tcp --port 3306 --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-07c328fdce581b30c",
            "GroupId": "sg-08808fb3017fe8ee8",
            "GroupOwnerId": "526680641953",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 3306,
            "ToPort": 3306,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
​
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 authorize-security-group-ingress --group-id sg-08808fb3017fe8ee8 --protocol tcp --port 22 --source-group sg-072fbdd8f5c28ab66
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0135f5ec03f8e4ef2",
            "GroupId": "sg-08808fb3017fe8ee8",
            "GroupOwnerId": "526680641953",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "ReferencedGroupInfo": {
                "GroupId": "sg-072fbdd8f5c28ab66",
                "UserId": "526680641953"
            }
        }
    ]
}
​
````
​
Now we are going to Launch 3 instances using keypair and the respective security groups:
​
Launching instance with only SSH access and it's name is "zomato-bastion"
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 run-instances --image-id ami-0beaa649c482330f7 --instance-type t2.micro --count 1 --key-name MyKeyPair --security-group-ids sg-072fbdd8f5c28ab66 --subnet-id subnet-028a6d96fc96a387b --associate-public-ip-address
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0beaa649c482330f7",
            "InstanceId": "i-09d5a77e117a4ee5e",
            "InstanceType": "t2.micro",
            "KeyName": "MyKeyPair",
            "LaunchTime": "2022-12-09T14:57:57+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-16-89-170.us-east-2.compute.internal",
            "PrivateIpAddress": "172.16.89.170",
-------------------------------------------------------
````
​
Then launch an instance with only php and httpd content installed and it's name is zomato-frontend
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 run-instances --image-id ami-0beaa649c482330f7 --instance-type t2.micro --count 1 --key-name MyKeyPair --security-group-ids sg-0537d3a2d2f41e2eb --subnet-id subnet-07c95b8c2995dd6ef --associate-public-ip-address
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0beaa649c482330f7",
            "InstanceId": "i-060142ee6d0751f9b",
            "InstanceType": "t2.micro",
            "KeyName": "MyKeyPair",
            "LaunchTime": "2022-12-09T14:56:45+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2a",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-16-59-29.us-east-2.compute.internal",
            "PrivateIpAddress": "172.16.59.29",
----------------------------------------------------------
````
​
Last I am going to launch an instance which have only database content installed and it's name is zomato-backend.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 run-instances --image-id ami-0beaa649c482330f7 --instance-type t2.micro --count 1 --key-name MyKeyPair --security-group-ids sg-08808fb3017fe8ee8 --subnet-id subnet-0efb3c7d55f3662ac --associate-public-ip-address
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0beaa649c482330f7",
            "InstanceId": "i-065ca84cb43a24236",
            "InstanceType": "t2.micro",
            "KeyName": "MyKeyPair",
            "LaunchTime": "2022-12-09T14:59:02+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2c",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-16-130-153.us-east-2.compute.internal",
            "PrivateIpAddress": "172.16.130.153",
     -------------------------------------------------------------
````
​
Now we launched 3 instances with the respective security groups. To confirm the same, I am going to describe to the instances to display the instance ID, Availability Zone, and the value of the Name tag for instances that have a tag with the name tag-key, in table format.
​
````
[ec2-user@ip-172-31-44-84 ~]$ aws ec2 describe-instances  --filters Name=tag-key,Values=Name --query 'Reservations[*].Instances[*].{Instance:InstanceId,AZ:Placement.AvailabilityZone,Name:Tags[?Key==`Name`]|[0].Value}' --output table
----------------------------------------------------------
|                    DescribeInstances                   |
+------------+-----------------------+-------------------+
|     AZ     |       Instance        |       Name        |
+------------+-----------------------+-------------------+
|  us-east-2a|  i-060142ee6d0751f9b  |  zomato-frontend  |
|  us-east-2b|  i-09d5a77e117a4ee5e  |  zomato-bastion   |
|  us-east-2c|  i-065ca84cb43a24236  |  zomato-backend   |
+------------+-----------------------+-------------------+
​
````
Thus we have created instances and assigned security groups to respective instances successfully.
​
Then, we are going to download and install wordpress in instance called "zomato-frontend" and database in "zoamto-backend". We can SSH to both the instances via "zomato-bastion".  Once it has been done, we can check whether the site is loading fine by accessing the site with the public DNS name or public IP of the "zomato-frontend". If the site running without issues, our setup has been completed.
​
Hope this will definitely help you to setup a simple wordpress setup with VPC and subnets and how to execute the same using AWSCLI command...Try same and have fun!!!!