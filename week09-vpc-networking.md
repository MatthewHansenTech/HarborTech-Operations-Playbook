# Week 9: VPC Networking and Troubleshooting

## HarborTech Ticket Summary
HarborTech received ticket TKT-2026-0009 about a web application that was not accessible from the internet, even though the EC2 web server was running and had a public IP address. This investigation reviewed the network path to determine whether routing, gateway configuration, security controls, or subnet design prevented users from reaching the website.

## Client Impact
Incorrect routing, subnet configuration, or traffic filtering can prevent users from accessing any web applications. When network traffic cannot reach a server, customers may be unable to access services, resulting in lost business, reduced productivity, and potential downtime. Even when a server is running, misconfigured routes, security groups, or network ACLs can block communication and make an application unavailable.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Amazon VPC: Used to review the network where the web server was deployed
- Subnets: To identify where the web server and private host were located
- Route Tables: Determine how network traffic was being routed
- Internet Gateway: Was checked to verify that internet access was available to the VPC
- NAT Gateway: Determines whether private resources had internet access
- Security Groups: Reviewed to verify that HTTP and SSH traffic were allowed
- Network ACLs: Identified any rules that could block network traffic
- CIDR Blocks: Reviewed to verify network addressing and subnet ranges
- Public and Private Subnets: Understood how resources were separated within the VPC
- VPC Flow Logs: To determine whether network traffic logs were available for troubleshooting

## Virtualization Connection
I learned that virtual servers communicate through virtual networks that use subnets, route tables, gateways, and security controls to manage traffic flow. Like traditional networking, virtualized cloud resources need proper routing and access controls to communicate successfully. A virtual machine can be running and still be unreachable if networking components are misconfigured.

## Evidence Reviewed

Command:
aws ec2 describe-vpcs

Observed Result:
"VpcId": "vpc-074d5ec0d39296a55",
"CidrBlock": "10.0.0.0/16",

Was performed in VPC vpc-074d5ec0d39296a55 using the 10.0.0.0/16 network range.

### Subnet CIDR

Command:
aws ec2 describe-subnets

Observed Result:
"SubnetId": "subnet-01a02c43e31da2485",
"CidrBlock": "10.0.1.0/24",

"SubnetId": "subnet-0419a775eb9330c4e",
"CidrBlock": "10.0.3.0/24",

The web server was located in the 10.0.1.0/24 subnet, while the private host was located in the 10.0.3.0/24 subnet.

### Subnet Type

Command:
aws ec2 describe-subnets

Observed Result:
"Value": "VPC1 Public Subnet 1",
"SubnetId": "subnet-01a02c43e31da2485",

"Value": "VPC1 Private Subnet 1",
"SubnetId": "subnet-0419a775eb9330c4e",

The web server was deployed in the public subnet, and the private host was deployed in the private subnet.

### Route Table Entries

Command:
aws ec2 describe-route-tables

Observed Result:
"RouteTableId": "rtb-0e316fc148cf742f1",

"GatewayId": "local",
"DestinationCidrBlock": "10.0.0.0/16",

The route table contained only the local route, with no default route 0.0.0.0/0 pointing to an Internet Gateway.

### Internet Gateway Attachment

Command:
aws ec2 describe-internet-gateways

Observed Result:
"InternetGatewayId": "igw-09d9fa30e71a757c5",

"Attachments": [
{
"State": "available",
"VpcId": "vpc-074d5ec0d39296a55"
}

The Internet Gateway was attached and available, but the route table was not directing traffic to it.

### NAT Gateway Configuration

Command:
aws ec2 describe-nat-gateways

Observed Result:
{
    "NatGateways": []
}

No NAT Gateway was configured within the environment.

### Security Group Rules

Command:
aws ec2 describe-security-groups

Observed Results:
"FromPort": 80,
"ToPort": 80,
"CidrIp": "0.0.0.0/0"

"FromPort": 22,
"ToPort": 22,
"CidrIp": "0.0.0.0/0"

Our security group allowed inbound HTTP and SSH traffic from any IP address.

### Network ACL Rules

Command:
aws ec2 describe-network-acls

Observed Result:
"RuleNumber": 40,
"Protocol": "6",
"From": 22,
"To": 22,
"RuleAction": "deny",
"CidrBlock": "0.0.0.0/0"

Rule 40 denied inbound SSH traffic on TCP port 22 from all IP addresses.

### Public and Private IP Addressing

Command:
aws ec2 describe-instances

Observed Result:
"PublicIpAddress": "98.83.29.163",
"PrivateIpAddress": "10.0.1.8",

The Web Server had both public and private IP addresses assigned.

### Application Port

Command:
aws ec2 describe-security-groups

Observed Result:
"FromPort": 80,
"ToPort": 80,
"CidrIp": "0.0.0.0/0"

Interpretation:
TCP port 80 was configured to allow HTTP traffic to the web application.

### Flow Log Evidence

Command:
aws ec2 describe-flow-logs

Observed Result:
{
    "FlowLogs": []
}

No VPC Flow Logs were configured in the environment.

### Connectivity Test Results

Command:
curl -I --connect-timeout 5 http://98.83.29.163

Observed Result:
curl: (28) Failed to connect to 98.83.29.163 port 80 after 5000 ms: Timeout was reached

The website could not be reached through HTTP.

## Operational Analysis
My evidence shows that the Web Server was running and had both a public IP address (98.83.29.163) and a private IP address (10.0.1.8) assigned. This confirmed that the server itself was operational and had network addressing configured. The Current security group allowed inbound HTTP traffic on TCP port 80 and inbound SSH traffic on TCP port 22 from 0.0.0.0/0, so the security group was not the primary cause of the website connectivity issue. Our route table evidence showed that route table rtb-0e316fc148cf742f1 contained only the local route 10.0.0.0/16 and no default route 0.0.0.0/0 pointing to Internet Gateway igw-09d9fa30e71a757c5. Because internet traffic wasn't directed to the Internet Gateway, users couldn't reach the website even though the web server had a public IP address. The Internet Gateway was attached and available, so the gateway itself wasn't the issue. Instead, the missing route prevented traffic from reaching the gateway. The Network ACL contained Rule 40, which denied inbound SSH traffic on TCP port 22 from all IP addresses. This explained why SSH connection attempts could not be established. While this did not directly cause the website outage, it prevented administrators from remotely accessing the server for troubleshooting. Our connectivity tests supported these findings. The HTTP test timed out and was unsuccessful. Based on the evidence collected, the primary issue involved routing, while the Network ACL created an additional administrative access issue.

## Recommendation
Based on the evidence I collected, I recommend adding a default route 0.0.0.0/0 to route table rtb-0e316fc148cf742f1 and directing all traffic to the Internet Gateway igw-09d9fa30e71a757c5. This recommendation is supported by the route table evidence and directly addresses the website accessibility issue reported in ticket TKT-2026-0009. I also recommend reviewing Network ACL acl-06ab14cefcbc219ed and modifying or removing Rule 40 so it no longer denies SSH traffic on TCP port 22. This would restore administrative connectivity without requiring changes to the VPC, subnet design, or security groups.

## Escalation Notes
The evidence showed that the recommended changes involve modifying route table rtb-0e316fc148cf742f1 and reviewing Network ACL acl-06ab14cefcbc219ed. Because routing and Network ACL changes affect network connectivity and access controls, the organization should review and approve these changes through its normal change-management process before implementation. My investigation did not identify any changes to the VPC design, subnet structure, security groups, or application configuration. Escalation would be required only if connectivity issues continue after the recommended routing and Network ACL changes are reviewed and implemented.

## Lessons Learned
This week taught me that a running EC2 instance with a public IP address doesn't automatically mean a website is reachable from the internet. Route tables, Internet Gateways, security groups, and Network ACLs all work together to create the network path. I also learned that troubleshooting should be based on evidence, not assumptions. By reviewing the VPC configuration, subnet configuration, route tables, Internet Gateway attachment, security groups, Network ACLs, and connectivity test results, I identified where the network path was failing and recommended changes that directly addressed the problem. Another important lesson was understanding the difference between security groups and Network ACLs. Even though the security group allowed HTTP and SSH traffic, the Network ACL still denied SSH access. This showed how multiple layers of security can affect connectivity and why you should review every layer during an investigation.

## Professional Vocabulary
Define the important Week 9 terms in your own words.
Include terms such as:
- VPC: A private network in AWS where you place and manage your cloud resources
- Subnet: A small section inside a VPC used to organize resources and traffic
- Route Table: A list of rules that tells network traffic where it should go
- Internet Gateway: A service that allows resources in a VPC to communicate with the internet
- NAT Gateway: A service that lets resources in a private subnet access the internet without allowing internet traffic directly back in
- Security Group: A virtual firewall that controls what traffic is allowed in and out of a resource
- Network ACL: An extra layer of network security that allows or blocks traffic at the subnet level
- CIDR: A range of IP addresses for a network
- Public Subnet: A subnet that can communicate directly with the internet through an Internet Gateway
- Private Subnet: A subnet that does not allow direct internet access and is usually used for internal resources
