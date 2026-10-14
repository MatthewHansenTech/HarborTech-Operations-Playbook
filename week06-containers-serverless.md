# Week 6: Containers and Serverless Computing

## HarborTech Ticket Summary
Bright Path Community Services reported that their current infrastructure includes a daily reporting task and a small REST API sharing a single EC2 instance that spends most of its time idling. HarborTech evaluates workload evidence across virtual machine, container, and serverless execution models to eliminate resource waste and optimize operational responsibility.

## Client Impact
An inefficient or mismatched compute model directly impacts client operations by incurring unnecessary costs for idle compute capacity and creating potential scalability or reliability bottlenecks during intermittent traffic surges.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Containers: Isolated application environments that package code and dependencies while sharing the host operating system kernel.
- Serverless Computing: A cloud execution model where the provider dynamically manages server infrastructure and resource allocation.
- AWS Lambda: Serverless compute service that executes code in response to automated triggers without provisioning servers.
- Amazon API Gateway: A fully managed service that handles secure public HTTP request routing, endpoint exposure, and traffic management.
- AWS Step Functions: A visual workflow service utilized to coordinate multiple serverless services.
- IAM Execution Roles: Security constructs and role permissions that grant serverless functions authorization to interact with AWS resources.
- Event-Driven Computing: Workloads that execute asynchronously in response to schedules, triggers, or cloud events.
- VPC-Connected Lambda: Network configurations that enable serverless functions to securely access internal resources within a private Virtual Private Cloud
- Monitoring and Logs: To track function execution duration, invocation logs, and error rates.

## Virtualization Connection
Containers and serverless computing move further away from traditional virtual machines by reducing management tasks. Virtual machines give you a full guest operating system on top of physical hardware; containers isolate your app by sharing the host operating system kernel; and serverless computing removes server and OS management from your daily tasks.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Workload duration and frequency reporting: the task executes precisely once each morning, while the REST API receives intermittent requests throughout the day
- Current server utilization metrics: confirm the shared EC2 instance remains lightly utilized and idle for prolonged periods
- Trigger or event pattern: daily report relies on a scheduled, event-driven trigger rather than continuous operation
- Runtime and dependency: workloads require standard application runtimes that can be decoupled from a permanent guest operating system
- IAM permissions: roles and permission boundaries that must be scoped to control service-to-service access securely
- API request behavior: traffic arrives intermittently, making a managed front-end routing service ideal
- Lambda logs and errors: execution logs and error tracking will be observed via monitoring tools post-migration
- VPC connectivity requirements: internal resource communication paths must be evaluated for secure service integration
- Workflow steps and failure points: must be verified to isolate code errors from integration failures
- Monitoring evidence: performance metrics and duration logs serve as the baseline for verifying function health

## Operational Analysis
Evidence from the current single-instance architecture shows that forcing continuous virtual machine operation for intermittent, event-driven workloads adds unnecessary operational and financial overhead. The daily reporting task runs once each morning and keeps no local state; running a full server all day wastes resources. Our small REST API experiences intermittent traffic rather than continuous. 
Given this workload behavior, the daily reporting task is best suited to an event-driven AWS Lambda, and the REST API is best suited to Amazon API Gateway paired with AWS Lambda. Moving these workloads away from a traditional virtual machine eliminates idle waste and shifts server infrastructure management entirely to the cloud provider.

## Recommendation
We recommend a split-model optimization strategy to best fit the workload behaviors. Transition the daily reporting task to an AWS Lambda function triggered by a scheduled event, eliminating daily idle time. Re-architect the small REST API using Amazon API Gateway and AWS Lambda to handle intermittent request spikes dynamically.
This recommendation addresses the operational need by removing idle server waste and scaling resources automatically based on actual traffic. The responsibility tradeoff shifts hardware provisioning, host scaling, maintenance, and guest OS patching entirely to AWS, while HarborTech and the client retain ownership of application logic, code security, and IAM execution-role permissions.

## Escalation Notes
Because these changes affect production systems, involve final migration plans, set up API domain cutovers, adjust IAM permissions, and shift the architecture, they must go through formal review and approval from HarborTech management and the client's Change Advisory Board. As an operations intern, I've documented all the evidence and recommendations here, but I didn't make any actual changes in the live environment.

## Lessons Learned
Week 6 showed me that choosing a compute model isn't about picking the most popular cloud service. It comes down to how the workload behaves: its timing, whether it's event-driven, and whether it needs to store state. Moving away from traditional virtual machines to containers and serverless functions makes a huge difference in reducing idle waste. Plus, using tools like API Gateway and Lambda taught me how cleanly you can split responsibilities between the cloud provider and our own application logic.

## Professional Vocabulary
Define the important Week 6 terms in your own words.
Include terms such as:
- Container: An isolated package that bundles your application code and dependencies.
- Serverless Computing: A cloud model where the provider handles all server management, scaling, and provisioning automatically.
- AWS Lambda: A serverless compute service that runs your code in response to events without needing a permanent server.
- API Gateway: A managed service that acts as a secure front door for routing and handling web requests.
- Execution Role: An IAM role that gives your serverless functions or container tasks permission to access AWS services.
- Event Trigger: Automated schedule or system event that kicks off a serverless task
- Step Functions: Visual tool used to connect and coordinate multiple serverless functions into a workflow
- VPC-Connected Lambda: Setup that lets your serverless functions securely communicate with internal resources inside a private network
- Runtime: Specific environment and language version needed to execute your code.
- Abstraction: A cloud feature that hides complicated hardware and OS maintenance so you can worry about something in your code.
