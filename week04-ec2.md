# Week 4: EC2 Evidence Lab

## HarborTech Ticket Summary
A newly provisioned Apache web server on EC2 is failing external HTTP connectivity checks. Remote curl attempts to the public IP time out, though the instance state reports running in AWS.
Ticket ID: TKT-2026-0004
Target Instance ID: i-0527cdd8b87411954 harbortech-wk4-server
Security Group ID: sg-041a4f43cec87e155
AWS Account: 956152099107 (us-east-1)

## Client Impact
External users cannot reach the HarborTech web service receiving network timeouts curl: (28) Connection timed out when attempting to load the page over HTTP.

## Environment and Resource Names
VPC and Subnet: vpc-0a1b2c3d4e5f6789a and then subnet-0123456789abcdef0
AMI and Profile: Amazon Linux 2023 ami-0c101f26f147fa7fd | LabInstanceProfile
Current Public IPs: 98.93.93.72 (Initial), 54.162.210.45 (Post-Restart)

## AWS Documentation Evidence
Security Groups: Stateful firewalls enforced at the hypervisor level, operating on a default-deny ingress model.
IP Address Lifecycle: Auto-assigned public IPv4 addresses are dynamic and released back to AWS when the instance stops, while EBS root volumes persist.

## CloudShell Command Record
aws ec2 describe-security-groups --group-ids sg-041a4f43cec87e155 --query SecurityGroups[0].IpPermissions

Checks our current security groups by seeing which group IDs are connected to the security group

aws ec2 authorize-security-group-ingress --group-id sg-041a4f43cec87e155 --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 describe-security-groups --group-ids sg-041a4f43cec87e155 --query SecurityGroups[0].IpPermissions

curl --connect-timeout 5 [http://98.93.93.72](http://98.93.93.72)

## Baseline Evidence
curl: (28) Connection timed out after 5002 milliseconds

## Root-Cause Analysis
Security Group sg-041a4f43cec87e155 lacked an inbound rule for TCP port 80. Due to the default-deny security model, all incoming web traffic was blocked before reaching the instance


## Corrective Action
Executed the following command aws ec2 authorize-security-group-ingress to permit TCP port 80 traffic from 0.0.0.0/0 at the control-plane layer

## Verification Evidence
The security group rule output showed an active TCP port 80 rule from 0.0.0.0/0.
The HarborTech Week 4 EC2 Evidence Lab responded to our HTTP request

## IMDSv2 and Guest Evidence
sh-5.2$ systemctl status httpd Output: active running
sh-5.2$ curl http://localhost Output: HarborTech Week 4 EC2 Evidence Lab
sh-5.2$ TOKEN=$(curl -sS -X PUT -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" "[http://169.254.169.254/latest/api/token](http://169.254.169.254/latest/api/token)")
sh-5.2$ curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" [http://169.254.169.254/latest/meta-data/instance-id](http://169.254.169.254/latest/meta-data/instance-id)i-0527cdd8b87411954

## Stop/Start Lifecycle Test
We stopped and restarted i-0527cdd8b87411954 through Cloud Shell, and it showed us that the moment we restarted, our public IP changed from 98.93.93.72 to 54.162.210.45. Then the moment we input our curl http://54.162.210.45 it returned our web page successfully, providing us with EBS storage persistence. 

## Cleanup Evidence
Terminated instance i-0527cdd8b87411954 and verified that the status has been terminated.
Deleted Security Group sg-041a4f43cec87e155.

## Escalation and Change-Control Notes
No escalation was needed; rebuilding was not justified because the current OS and web application were healthy, but the issue was in the security group rule control plane. Control now stops any production instance, causing downtime and public IP changes.

## Lessons Learned
I learned that the control plane must verify and connect to the firewall rules, which must be separate from the OS service. Auto-assigned public IPv4 addresses can change completely when you stop and restart instances. I thought they would stay the same, but AWS documentation says production workloads need Elastic IPs or load balancers to remain consistent.

## Professional Vocabulary
EC2 Instance: A virtual server configured with specific compute, memory, and storage capacity
AMI: An image template an EC2 instance would use, containing the operating system, application server, and software configurations required to launch an instance
EBS: a network-attached block storage designed for use with EC2 instances that retains data independently
Security group: controls inbound and outbound traffic to instances using explicit permission rules
User Data: allows you to automate with the use of a script provisioning the config of a virtual machine during its boot cycle
Metadata: Operating data about a running instance, providing Instance ID, private/public IPs, and IAM roles
Lifecycle: The status of an instance, such as pending, running, or stopped
AWS Control Plane: A manager of the api layer used by administrators to configure, inspect, and route cloud infrastructure
Guest OS Layer: Going inside a running virtual machine
Default Deny Ingress: A security architecture where all incoming network traffic is blocked and must be manually allowed to pass through.

