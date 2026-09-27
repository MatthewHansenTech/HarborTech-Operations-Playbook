# Week 3: Systems Manager and S3

## HarborTech Ticket Summary
Ticket TKT-2026-0003 documents the operational escalation from Bright Path Community Services regarding inefficient, manual instance management and static site deployment. The organization currently relies on repeated manual administrative processes to manage all five EC2 instances and server infrastructure for the current static resource web pages, increasing operational overhead and configuration drift.

## Client Impact
Repeating manual server administration hurts consistency, efficiency, and support: Configurations are applied manually across multiple EC2 instances, causing configuration drift as server environments gradually diverge over time and making future troubleshooting unpredictable. For efficiency, we log into each virtual machine sequentially, consuming time that should be spent on higher-priority tasks. Lastly, support operations are manual and carry a higher risk of human error, creating auditing gaps and lengthening resolution times during operational incidents.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- AWS Systems Manager: A central management service for operational data and automated tasks across AWS resources.
- Run Command: AWS Systems Manager capability allowing remote, secure script execution on managed instances at scale without requiring SSH access.
- Session Manager: A fully managed capability providing secure, browser-based interactive shell access to instances without open inbound ports.
- Inventory: Collects operational metadata of current OS versions, installed applications, and network settings from managed instances.
- Parameter Store: A secure storage location for configuration data and secrets for database endpoints and connection strings.
- Amazon S3: A highly scalable object storage service designed for durability and availability.
- Static Website Hosting: An Amazon S3 configuration enabling direct serving of static client-side web assets of HTML, CSS, JS, and images without virtual web servers.

## Virtualization Connection
AWS Systems Manager acts as a centralized management layer by controlling virtual machines, EC2 instances, or on-premises servers. It lets administrators query, track, and update OS-level states uniformly, regardless of the underlying physical hardware or hypervisor layer. S3 supports workloads that don't require a traditional server by enabling static website hosting and creating a serverless architecture. Allowing us to create serverless object storage, replacing web servers like Apache running on our EC2 instance and serving client-side content. 

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Current manual administration steps: Sequential server logins are required for maintenance routines and static asset deployment.
- Number of affected systems: 5 EC2 instances requiring maintenance routines.
- Managed node requirements: Instances require the SSM Agent to be installed, an IAM instance profile attachment for AmazonSSMManagedInstanceCore, and any outbound SSM endpoint connectivity.
- Systems Manager feature fit: Run Command fits best with this scenario because it allows us to run a command across all 5 EC2 instances that can not require us to log in to each device manually, wasting our time. By just executing a script, it avoids the risk of human error but also provides us with a way to automatically update any instances that are using the same script.
- Interactive versus non-interactive access needs: The script execution requires non-interactive access using Systems Manager Run Command, which allows us to run a script across all instances; troubleshooting requires secure interactive shell sessions via Session Manager.
- Configuration or parameter needs: Application parameter needs are /brightpath/prod/db-endpoint, which is stored outside the hardcoded local files.
- Static content requirements: HTML pages include program hours, resource lists, and do not require any server-side code execution. 
- Evidence available through the AWS environment:
    1. Active IAM identity and account authorization verified via aws sts get-caller-identity.
    2. S3 bucket creation which is brightpath-resource-site-mh2026 confirmed the region is in us-east-1 via aws s3 mb
    3. Static website hosting properties successfully registered; suffix: index.html verified via aws s3api get-bucket-website.
    4. Public Access Block configurations set to false, confirmed via aws s3api get-public-access-block.
    5. Differential file sync and timestamp verification confirmed via aws s3 sync . s3://brightpath-resource-site-mh2026/.

## Operational Analysis
Centralized Management and Automation are best for tasks like batch software updates, patch management, and maintenance scripts across all five instances; use Systems Manager Run Command to eliminate repetitive labor and ensure atomic state consistency.
TKT-2026-0003 highlights configuration inefficiencies caused by managing 5 EC2 instances individually. We implement an SSM Run Command that runs scripts simultaneously across all managed nodes without opening and logging into each instance.
Use Interactive Access only for emergency troubleshooting with Session Manager rather than opening SSH/port 22 connections, improving security posture and audit logs.
Static Object Hosting is best for serving public information sites using S3 Static Website Hosting, which lowers operational costs, eliminates EC2 provisioning, and scales automatically under load.
We tested successful static site hosting using AWS S3 website hosting and a file sync with AWS S3 sync, allowing us to verify our endpoint, which responded at http://brightpath-resource-site-mh2026.s3-website-us-east-1.amazonaws.com, proving it serves static assets.

## Recommendation
AWS Systems Manager Run Command lets us run automated scripts across all five instances simultaneously without opening inbound port 22 or managing SSH keys.

Bright Path Web Deployment, we want to allow Amazon S3 Static Website Hosting, which delivers static content directly from object storage using aws s3 sync, removing the cost and complexity of maintaining web server compute instances.

## Escalation Notes

No escalations are required for the CLI operations completed in the Learner Lab sandbox, including caller identity verification, S3 bucket creation, static website hosting configuration, and public access block modifications.

## Lessons Learned
This week taught me that centralized management manages cloud nodes through a control plane, reducing complexity compared to host-by-host management. Safe automation taught me how to use multi-node command execution to reduce human error during deployment. Selecting the right service model helps us view the advantages of choosing server object storage over a compute-bound VM for static web workloads, reducing maintenance and improving reliability

## Professional Vocabulary
Define the important Week 3 terms in your own words.
Include terms such as:
- Systems Manager: Allows tracking, managing, and automating tasks across AWS cloud resources.
- Managed Node: Any EC2 instance or on-premises server configured with the SSM Agent and an IAM role allowing communication with Systems Manager.
- Run Command: An SSM capability used to remotely and securely run shell scripts or commands.
- Session Manager: An SSM capability providing interactive single-click browser or CLI shell access without the use of logging into systems.
- Inventory: Automatically gathers system configuration data, software, patches, and OS info.
- Parameter Store: A secure, centralized store for managing configuration parameters across applications.
- Automation: Simplifies the repetitive maintenance and deployment tasks through custom workflows
- Static Website Hosting: An Amazon S3 feature that serves static web files directly over HTTP/HTTPS without requiring a backend web server.
- Object Storage: Manages data as individual discrete objects within flat namespaces.
- Management Plane: An architectural layer handling administrative tasks and configuration enforcement to control requests.
