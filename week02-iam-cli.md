# Week 2: IAM and AWS CLI Investigation

## HarborTech Ticket Summary
Marcus Webb can authenticate with Riverside Goods' AWS account but cannot perform any assigned inventory work. We confirmed that Marcus has no permissions to upload within riverside-inventory. Marcus is currently the Inventory Coordinator and needs to read inventory, report objects, and approve inventory files. He has no onboarding record showing the IAM user exists, with no job-function group and no directly attached permission policy. 

## Client Impact
How missing or incorrect permissions affect Marcus Webb's ability to perform the required work: He cannot upload any objects to riverside-inventory. The business states his job is to maintain the inventory by reading inventory, reporting objects, and uploading approved files. If he lacks permission to do so, there is a huge risk because no one is managing the unupdated list in the bucket.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- AWS Identity and Access Management (IAM) - Authenticated users to provide a trusted work environment and the creation of users, groups, and roles
- IAM users, groups, roles, and policies - A restriction measure for accounts, creating a way to authorize users to do explicit permissions focused on configured policy documents
- Amazon S3 - Allows management of an object storage service to store and retrieve amounts of data
- AWS CloudShell - Grants us the ability to manage and navigate resources directly from the AWS management console
- AWS Command Line Interface (AWS CLI) A console to allow the viewing or inputting of commands to interact with the creation of resources within AWS
- AWS Regions - Puts you in an area close to a data center that can safely store your resources

## Virtualization Connection
Identity and access controls are who or what has the direct ability to modify, create, or delete any cloud resources 

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Successful authentication - Marcus was able to successfully sign in to the Riverside Goods AWS console with his assigned IAM user credentials
- Required job responsibilities - Marcus is the inventory coordinator, and his job responsibilities are to list the riverside-inventory bucket and be able to read inventory, report objects, and upload any approved inventory files
- IAM identity information - Marcus Webb AWS account
- Group or policy evidence - The onboarding record shows Marcus has no current job-function group and has no directly attached permission policies
- AccessDenied output - Access Denied when requesting to list inventory
- AWS CLI caller identity - "User ID "XXXXXXXXXXXXXXXXXXXXXXX:userXXXXXXX=Matthew_J._Hansen "Account" XXXXXXXXXXX "ARN" arn:aws;sts::XXXXXXXXXXXXXXXXXX:assumed-role/voclabs/userXXXXXXX=Matthew_J._Hansen
- Requested S3 actions - s3:ListBucket allows to view files s3:GetObject allows to read and report s3:PutOject to allow uploading
- Resource scope - arn:aws:s3:::riverside-inventory & arn:aws:s3:::riverside-inventory/*

## Operational Analysis
The evidence shows that Marcus currently has no authorization. Authentication and authorization are completely different: authentication verifies your credentials to log in, while authorization determines what you can do once you're logged in with the current role.

## Recommendation
An approach we can use is to create a managed policy that is attached to a dedicated job-function IAM group and apply permissions directly to a user

## Escalation Notes
We can create an IAM group with permissions above a standard user, allowing Marcus to view, report, create, and upload verified files. If this doesn't work, we can authorize specific actions within the bucket, which would be the best approach because we can ask the current script to include s3:PutObject, allowing Marcus to upload new inventory. Because currently is allowed to view inside the bucket and read 


## Lessons Learned
Week 2 taught me that IAM privileges aren't a simple on/off switch that gives you permission to do whatever. It's a step-by-step process to figure out what users can do, what they can't, and how to configure it properly. It also shows how to navigate CloudShell to view your account details, like your role, name, and anything that supports authentication. Least privilege ensures that users have only the permissions they need to do their job, and that access is granted through a defined process. The AWS CLI shows what a person enters in the command shell and can verify whether they entered the correct commands to receive the correct output. For my student authentication, I've bleeped out the special characters for security, but this verifies who I am, and I can see a ton of descriptions of what my current role's abilities are and what the console shows. When identifying access problems, I will run into them in the future since AWS Learner Lab has limits, but this week I had no difficulty accessing information. 

## Professional Vocabulary
Define the important Week 2 terms in your own words.
Include terms such as:
- Authentication - A way to identify your account that is you that is signing in
- Authorization - Access for certain permissions that allow or deny your access
- IAM - The identity of the user, with authentication, and current authorization for resources
- Policy - A set of rules that allows a user to follow and can allow or deny access
- Least Privilege - Gives a user the lowest permissions of access
- AccessDenied - You lack the authorization to complete the action
- AWS CLI - An interface that allows you to input commands within a command line shell to create, manage, or delete resources
- CloudShell - Allows you to manage multiple cloud resources without installing hardware
- Caller Identity - Provides authenticated information about the account, role, ID, and additional information about its current limits.
- Resource Scope - Boundaries for resources that are configured with rules
