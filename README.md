# AWS-CLI-Create-VPC-Subnets-Route-tables-Security_groups-Instance

Setting up a VPC (Virtual Private Cloud) interface using AWS CLI (Command Line Interface) is a straightforward process. The AWS CLI provides a command-line interface to interact with various AWS services, including VPC. Here's an overview of how you can set up a VPC interface using AWS CLI.
i'm going to create a VPC, subnets, route tables, security groups, keypair and instance.

The choice between AWS CLI and Terraform depends on your preference, the complexity of your infrastructure, and your team's expertise. If you prefer scripting and want fine-grained control, AWS CLI might be suitable. If you prefer a declarative and infrastructure-agnostic approach, Terraform can be a good choice. Some organizations even use a combination of both.


### Features of AWS CLI

- Command Line Access: AWS CLI provides a command-line interface to interact with various AWS services. It allows you to execute commands, manage resources, and automate tasks.
- Easy Installation and Updates: AWS CLI can be easily installed using package managers.


### Contents


   - Create a VPC
   - Create public and private subnets
   - Create internet gateway for the VPC
   - Create an elastic IP address for NAT gateway
   - Create a NAT gateway
   - Create a route table for each subnet
   - Create routes
   - Associate route table to subnet
   - Create a security group for the VPC
   - Create Key-pair
   - Run an instance

### Prerequisites

1. An AWS account.
2. Configure AWS CLI using an IAM user with programmatic access.

**You’re good to go. Now, let’s get started!**

### Step 1 — Create a VPC 

This command creates a VPC with the CIDR block range of 172.32.0.0/16. The CIDR block determines the IP address range that will be available for your VPC.

After executing the command, you will receive a JSON output containing information about the created VPC, including its VPC ID, CIDR block range, and other attributes.
```
$ aws ec2 create-vpc --cidr-block 172.32.0.0/16

{
    "Vpc": {
        "CidrBlock": "172.32.0.0/16",
        "DhcpOptionsId": "dopt-03970761f5ca105a9",
        "State": "pending",
        "VpcId": "vpc-05f118ed4ba7b9cc2",
        "OwnerId": "175601052213",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-00c7985d48e395bb7",
                "CidrBlock": "172.32.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false
    }
}
```
> <b>Add a name tag for VPC</b>
```
$ aws ec2 create-tags --resources vpc-05f118ed4ba7b9cc2 --tags Key=Name,Value=my_vpc
```

### Step 2 - Create public and private subnets 

><b> Create public subnet 1</b>
```
aws ec2 create-subnet --vpc-id vpc-05f118ed4ba7b9cc2 --cidr-block 172.32.0.0/18 --availability-zone ap-south-1a
{
    "Subnet": {
        "AvailabilityZone": "ap-south-1a",
        "AvailabilityZoneId": "aps1-az1",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.32.0.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-06f7ef739b0430039",
        "VpcId": "vpc-05f118ed4ba7b9cc2",
        "OwnerId": "175601052213",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:ap-south-1:175601052213:subnet/subnet-06f7ef739b0430039"
    }
}
```
> <b> Add a tag to the subnet</b>
```
$  aws ec2 create-tags --resources subnet-06f7ef739b0430039 --tags Key=Name,Value=Subnet1
```
> <b>Create public subnet 2</b>
```
$ aws ec2 create-subnet --vpc-id vpc-05f118ed4ba7b9cc2 --cidr-block 172.32.64.0/18 --availability-zone ap-south-1b
{
    "Subnet": {
        "AvailabilityZone": "ap-south-1b",
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.32.64.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0bdeb0a332f629cd5",
        "VpcId": "vpc-05f118ed4ba7b9cc2",
        "OwnerId": "175601052213",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:ap-south-1:175601052213:subnet/subnet-0bdeb0a332f629cd5"
    }
}
```
> <b>Add a tag to the subnet</b>
```
$ aws ec2 create-tags --resources subnet-0bdeb0a332f629cd5 --tags Key=Name,Value=Subnet2
```
> <b> Create a private subnet</b>
```
$ aws ec2 create-subnet --vpc-id vpc-05f118ed4ba7b9cc2 --cidr-block 172.32.128.0/18 --availability-zone ap-south-1b
{
    "Subnet": {
        "AvailabilityZone": "ap-south-1b",
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.32.128.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-041589df7acf7d39e",
        "VpcId": "vpc-05f118ed4ba7b9cc2",
        "OwnerId": "175601052213",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:ap-south-1:175601052213:subnet/subnet-041589df7acf7d39e"
    }
}
```
> <b> Add a tag to subnet</b>
```
$ aws ec2 create-tags --resources subnet-041589df7acf7d39e --tags Key=Name,Value=Subnet3
```
> <b>Enable "Auto-assign Public IP" for two public subnets</b>
```
$ aws ec2 modify-subnet-attribute --subnet-id subnet-0bdeb0a332f629cd5 --map-public-ip-on-launch
$ aws ec2 modify-subnet-attribute --subnet-id subnet-06f7ef739b0430039 --map-public-ip-on-launch
```
### Step 3 - Create internet gateway for the VPC 

Internet gateway (IGW) is simply used to connect a VPC to the internet so that the VPC resources can access the internet and be accessed over the internet.

```
$ aws ec2 create-internet-gateway
{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-0bd36620eac2625ec",
        "OwnerId": "175601052213",
        "Tags": []
    }
}
```
> <b>Add a tag </b>
```
$ aws ec2 create-tags --resources igw-0bd36620eac2625ec --tags Key=Name,Value=igw
```
><b>Attach igw to VPC</b>
```
$ aws ec2 attach-internet-gateway --internet-gateway-id igw-0bd36620eac2625ec --vpc-id vpc-05f118ed4ba7b9cc2
```

### Step 4 - Create an elastic IP address for NAT gateway
```
$ aws ec2 allocate-address --domain vpc
{
    "PublicIp": "3.111.5.15",
    "AllocationId": "eipalloc-0692e65c6d29bdd6a",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "ap-south-1",
    "Domain": "vpc"
}
```
###  Step 5 - Create a NAT gateway 

NAT Gateway is a managed AWS service that allows instances in private subnets to access the internet while preventing inbound connections from the internet.
```
$ aws ec2 create-nat-gateway --subnet-id subnet-06f7ef739b0430039 --allocation-id eipalloc-0692e65c6d29bdd6a
{
    "ClientToken": "38d2064b-b0b3-45d1-b41b-751eb56eb996",
    "NatGateway": {
        "CreateTime": "2023-05-25T11:17:43.000Z",
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-0692e65c6d29bdd6a"
            }
        ],
        "NatGatewayId": "nat-0618c0e7b52dc0a32",
        "State": "pending",
        "SubnetId": "subnet-06f7ef739b0430039",
        "VpcId": "vpc-05f118ed4ba7b9cc2"
    }
}
```
><b>Add a tag to the NAT gw</b>
```
$ aws ec2 create-tags --resources nat-0618c0e7b52dc0a32 --tags Key=Name,Value=nat-gw
```

###  Step 6 - Create a route table for each subnet 
><b>Create rtb 1</b>
```
$ aws ec2 create-route-table --vpc-id vpc-05f118ed4ba7b9cc2
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-0cb37c83c5c185023",
        "Routes": [
            {
                "DestinationCidrBlock": "172.32.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [],
        "VpcId": "vpc-05f118ed4ba7b9cc2",
        "OwnerId": "175601052213"
    }
}
```
><b>Add a tag to the public rtb</b>
```
$ aws ec2 create-tags --resources rtb-0cb37c83c5c185023 --tags Key=Name,Value=public
```
><b>Create rtb 2</b>
```
$ aws ec2 create-route-table --vpc-id vpc-05f118ed4ba7b9cc2
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-0c0fadfad1bda183a",
        "Routes": [
            {
                "DestinationCidrBlock": "172.32.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [],
        "VpcId": "vpc-05f118ed4ba7b9cc2",
        "OwnerId": "175601052213"
    }
}
```
><b>Add a tag to private rtb</b>
```
$ aws ec2 create-tags --resources rtb-0c0fadfad1bda183a --tags Key=Name,Value=private
```

### Step 7 - Create routes 

Attach the route table created for the public subnet to the internet gateway. The route matches all IPv4 traffic (0.0.0.0/0) and routes it to the specified Internet gateway.
```
$ aws ec2 create-route --route-table-id rtb-0cb37c83c5c185023 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0bd36620eac2625ec{
    "Return": true
}
```
Attach the route table created for the private subnet to the NAT gateway. The route matches all IPv4 traffic (0.0.0.0/0) and routes it to the specified NAT gateway.
```
$ aws ec2 create-route --route-table-id rtb-0c0fadfad1bda183a --destination-cidr-block 0.0.0.0/0 --gateway-id nat-0618c0e7b52dc0a32{
    "Return": true
}
```
### Step 8 - Associate route table to subnets
><b>Associate the public route table to the public subnet1</b>
```
$ aws ec2 associate-route-table --route-table-id rtb-0cb37c83c5c185023 --subnet-id subnet-0bdeb0a332f629cd5
{
    "AssociationId": "rtbassoc-07ae4f7434bb6d0f8",
    "AssociationState": {
        "State": "associated"
    }
}
```
><b>Associate the public route table to the public subnet2</b>

```
$ aws ec2 associate-route-table --route-table-id rtb-0cb37c83c5c185023 --subnet-id subnet-06f7ef739b0430039
{
    "AssociationId": "rtbassoc-0671f3b6f44450b71",
    "AssociationState": {
        "State": "associated"
    }
}
```
><b>Associate the private route table to the private subnet</b>

```
$ aws ec2 associate-route-table --route-table-id rtb-0c0fadfad1bda183a --subnet-id subnet-041589df7acf7d39e
{
    "AssociationId": "rtbassoc-0ea100022bebf2ef2",
    "AssociationState": {
        "State": "associated"
    }
}
```
###  Step 9 - Create a security group for the VPC 

```
$ aws ec2 create-security-group --group-name aws-sg  --description "An sg for traffic control" --vpc-id vpc-05f118ed4ba7b9cc2
{
    "GroupId": "sg-05dd2bee38394b8fb"
}
```
><b>Add a tag to the SG</b>
```
$ aws ec2 create-tags --resources sg-05dd2bee38394b8fb --tags Key=Name,Value=aws-sg
```
><b>Specify rules for the security group</b>
```
$ aws ec2 authorize-security-group-ingress --group-id sg-05dd2bee38394b8fb --protocol tcp --port 22 --cidr 0.0.0.0/0

$ aws ec2 authorize-security-group-ingress --group-id sg-05dd2bee38394b8fb --protocol tcp --port 80 --cidr 0.0.0.0/0
```

### Step 10 - Create Key-pair
```
$ aws ec2 create-key-pair --key-name aws-keyPair --query 'KeyMaterial' --output text > aws-keyPair.pem
```

### Step 11 - Create an instance

```
$ aws ec2 run-instances --image-id ami-0c768662cc797cd75 --instance-type t2.micro --count 1 --subnet-id subnet-06f7ef739b0430039 --security-group-ids sg-05dd2bee38394b8fb --associate-public-ip-address --key-name aws-keyPair
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0c768662cc797cd75",
            "InstanceId": "i-0d7516225b5ac71af",
            "InstanceType": "t2.micro",
            "KeyName": "aws-keyPair",
            "LaunchTime": "2023-05-25T11:40:42.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "ap-south-1a",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-32-61-20.ap-south-1.compute.internal",
            "PrivateIpAddress": "172.32.61.20",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-06f7ef739b0430039",
            "VpcId": "vpc-05f118ed4ba7b9cc2",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "cccf80ba-d4df-45a8-ae26-74ef7151c69e",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-05-25T11:40:42.000Z",
                        "AttachmentId": "eni-attach-0e5ec4fb6b4afc9ba",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "aws-sg",
                            "GroupId": "sg-05dd2bee38394b8fb"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "02:ae:53:dd:76:d0",
                    "NetworkInterfaceId": "eni-07f1ac538e6da57c1",
                    "OwnerId": "175601052213",
                    "PrivateIpAddress": "172.32.61.20",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateIpAddress": "172.32.61.20"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-06f7ef739b0430039",
                    "VpcId": "vpc-05f118ed4ba7b9cc2",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "aws-sg",
                    "GroupId": "sg-05dd2bee38394b8fb"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "required",
                "HttpPutResponseHopLimit": 2,
                "HttpEndpoint": "enabled"
            }
        }
    ],
    "OwnerId": "175601052213",
    "ReservationId": "r-0d2d8dbd60b0e4857"
}
```
><b>Add a tag to the instance</b>
```
$ aws ec2 create-tags --resources i-0d7516225b5ac71af --tags Key=Name,Value=webserver
```
><b>Enable the DNS hostname for the VPC</b>

```
$ aws ec2 modify-vpc-attribute --vpc-id vpc-05f118ed4ba7b9cc2 --enable-dns-hostnames
```
><b>Print Public DNS name</b>

```
$ aws ec2 describe-instances --instance-ids i-0d7516225b5ac71af --query 'Reservations[].Instances[].PublicDnsName'
[
    "ec2-13-234-202-221.ap-south-1.compute.amazonaws.com"
]
```

As for comparing AWS CLI and Terraform, they serve different purposes. AWS CLI is a command-line tool provided by AWS for managing AWS resources directly through commands. It provides more flexibility but requires manual scripting and maintenance.

On the other hand, Terraform is an infrastructure as code (IaC) tool that allows you to define and manage your infrastructure using declarative configuration files. Terraform provides a higher-level abstraction and automates the creation and management of resources across multiple cloud providers, including AWS. It offers versioning, state management, and can be used for orchestration.

