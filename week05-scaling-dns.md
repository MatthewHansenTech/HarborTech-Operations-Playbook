# Week 5: Scaling, Load Balancing, and DNS

## HarborTech Ticket Summary
HarborTech Support Ticket TKT-2026-0005 documents the upcoming high-volume traffic surge for client Riverside Goods. The application currently operates only on a single EC2 instance connected to a single endpoint. The primary problem is limited capacity and poor availability; while no outages have occurred yet, the current setup cannot dynamically scale or distribute incoming client requests during peak loads.

## Client Impact
If a traffic surge hits the single EC2 instance without operational changes, the server's CPU will reach 100% utilization. This will result in dropped requests, application timeouts, and total service disruption for all Riverside Good end users. For business operations, this creates direct revenue and customer-trust losses, plus unhandled operational failures due to a single point of failure.

## Provided Ticket Evidence
Ticket ID: TKT-2026-0005 Traffic Surge - Design the Operational Response
Target Environment State: Riverside Goods application running on a single standalone EC2 instance
Performance: Application endpoint experiencing up to 100% CPU utilization under simulated traffic surge conditions
Architecture Limits: No active Elastic Load Balancing, Auto Scaling Groups, or Route 53 DNS failover configured before intervention.

## AWS Commands Used
Auto Scaling Groups
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' \
  --output table

Target Groups
aws elbv2 describe-target-groups \
  --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' \
  --output table

Target Health Check Audit
TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups --query 'TargetGroups[0].TargetGroupArn' --output text)
echo "$TARGET_GROUP_ARN"
aws elbv2 describe-target-health \
  --target-group-arn "$TARGET_GROUP_ARN" \
  --query 'TargetHealthDescriptions[].{Target:Target.Id,Port:Target.Port,State:TargetHealth.State,Reason:TargetHealth.Reason}' \
  --output table

Route 53 DNS Routing
aws route53 list-hosted-zones \
  --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' \
  --output table

## AWS Evidence Collected
Auto Scaling Group Output: [] Blank table output   

Target Group ARN Query: None

Target Health Output: An error occurred "ValidationError" when calling the DescribeTargetHealth operation: 'None' is not a valid target group ARN

Route 53 Hosted Zones Output: [] Blank table output

## Virtualization Connection
Virtualization turns physical hardware into software-based virtual servers. Because virtual servers aren't tied to one physical computer, scaling and load balancing work together like a traffic system. A load balancer distributes incoming requests evenly across multiple virtual servers. If one server fails a health check, traffic is automatically redirected away from it so users never notice an issue. When CPU usage spikes during a traffic spike, auto scaling spins up a new virtual server to help share the workload. If traffic calms down, auto scaling removes those extra virtual servers so you only pay for what you needed during the spike. 

## Operational Analysis
Ticket evidence from AWS shows that Riverside Goods currently operates without a high-availability infrastructure. We receive a blank output from describe-auto-scaling-groups, which tells us no automated capacity management exists, meaning the server cannot scale when CPU usage spikes. For any target health and traffic distribution, we used the Target Group ARN, which showed None, confirming no Application Load Balancer targets exist. Traffic flows directly to the single instance public IP, with no load balancing or automated health checks. Lastly, our list of hosted zones is empty, confirming Route 53 is not configured to manage any DNS records or perform automated endpoint failover.

## Recommendation
We need to eliminate all single points of failure while maintaining costs efficiently. Deploy a multi-AZ Auto Scaling Group with a minimum of 2, a desired of 2, and a Maximum of 6 instances, using a dynamic scaling policy triggered at 70% CPU utilization. Implement an application load balancer and associate it with a target group with HTTP health checks to evenly distribute incoming client traffic across active, healthy instances. The final recommendation is to configure a public hosted zone with failover routing to shift traffic away from degrading primary endpoints to a secondary recovery source. 

## Escalation Notes
Management & Budget Approval Required: Executive management must approve increased operational expenditure for running multi-AZ EC2 instances in a 2/2/6 configuration and active ALB resources.
Architecture Team Approval Required: Cloud Architecture review and sign-off are required prior to modifying production networking paths, deploying new load balancers, and altering public DNS records.

## Lessons Learned
Evidence-Based Auditing taught me that empty CLI outputs [] and validation errors (None) are not system failures; they are critical diagnostic evidence of missing infrastructure baseline configurations. Relying on a single compute instance creates an unavoidable single point of failure during traffic spikes, making elasticity essential for operational resilience. Load balancers depend entirely on active health checks to determine whether backend targets are healthy/unhealthy before routing client traffic. High availability also requires DNS-level routing strategies like Route 53 failover to redirect traffic seamlessly when primary endpoints become unreachable or degraded. Scaling out compute capacity must always be weighed against financial constraints and operational budget approvals.

## Professional Vocabulary
Elasticity: The ability of cloud infrastructure to automatically expand or shrink compute capacity as real-time demand fluctuates.

Scalability: An architecture's structural capacity to handle increased workload volumes by adding resources.

Load Balancer: A service that distributes incoming application traffic evenly across multiple backend instances to ensure high performance and prevent single-server overload.

Target Group: A designated group of backend compute resources, such as EC2 instances, registered to receive routed requests from a load balancer.

Health Check: An automated verification check a load balancer sends periodically to confirm an instance is operational.

Auto Scaling Group: A collection of EC2 instances that automatically provisions or terminates instances based on demand policies or health status.

Launch Template: A saved configuration blueprint containing instance details—such as AMI ID, instance type, security groups, and key pairs.

Desired Capacity: The target baseline count of active instances an Auto Scaling Group maintains during regular operations.

Route 53: AWS cloud Domain Name System that translates user domain names into IP addresses and routes internet traffic.

Failover: An automated operational mechanism that redirects client traffic from an unhealthy primary endpoint to a designated secondary operational endpoint.     
