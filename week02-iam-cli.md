# Week 2: IAM and AWS CLI Investigation

## HarborTech Ticket Summary
HarborTech Support received a ticket TKT-2026-0002 regarding Marcus Webb, an Inventory Coordinator for Riverside Goods. The ticket reports that Marcus can successfully authenticate and log into the AWS Management Console, but he receives an "AccessDenied" error whenever he attempts inventory management tasks on the assigned S3 bucket. HarborTech was tasked with diagnosing whether the issue stems from an authentication failure or an authorization gap, evaluating a client access request, and recommending a least-privilege access model.

## Client Impact
Because of the restriction, Marcus cannot view inventory reports, list bucket contents, or upload approved inventory documentation to Amazon S3. This completely blocks his ability to perform daily inventory tracking for Riverside Goods, risking inventory miscounts and operational delays.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- AWS Identity and Access Management (IAM): Used to manage user identities, create roles, trust policies, and permission boundaries.
- IAM users, groups, roles, and policies: The core identity components using JSON policy documents controlling authorization.
- Amazon S3: Object storage service hosting within the riverside-inventory bucket.
- AWS CloudShell: Browser-based shell environment used to securely execute CLI commands in the AWS Console.
- AWS Command Line Interface (AWS CLI): Tool used to identify contents and inspect IAM entities.
- AWS Regions: Locations hosting the isolated S3 storage resources and IAM control plane endpoints.

## Virtualization Connection
In software-defined cloud infrastructures, identity-based access controls replace physical perimeters. Just as hypervisors control access to virtualized compute, networking, and storage resources, AWS IAM acts as the control-plane perimeter. Without authorization rules, unauthorized identities could modify software-defined networks, alter virtual storage permissions, and disrupt cloud workloads.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Successful authentication: Marcus signed into the AWS Console using valid IAM user credentials (Evidence A).
- Required job responsibilities: Daily workflow required for listing bucket contents, downloading report objects, and uploading new inventory files (Evidence B).
- IAM identity information: Marcus operates as an IAM user identity within the client's AWS account.
- Group or policy evidence: Account inspection confirmed zero attached managed/inline policies and no active IAM group memberships (Evidence C).
- AccessDenied output: Requesting bucket contents returned an explicit "AccessDenied" error response (Evidence D).
- AWS CLI caller identity: Running "aws sts get-caller-identity" confirmed active identity context "arn:aws:sts::*************:assumed-role/voclabs/user*******=Username".
- Requested S3 actions: Required API calls are "s3:ListBucket", "s3:GetObject", and "s3:PutObject".
- Resource scope: Targets are bucket "arn:aws:s3:::riverside-inventory" and objects "arn:aws:s3:::riverside-inventory/*".

## Operational Analysis
The evidence shows that the access problem is an authorization gap, and not an authentication failure.
Authentication was verified when Marcus successfully signed in with valid credentials
Authorization failed because AWS IAM operates under a default-deny evaluation mode. Because Marcus has no attached permission policies or group memberships, AWS denied the S3 API calls by default.

## Recommendation
Reject "AmazonS3FullAccess" the client requested and implement a customer-managed, least-privilege IAM policy attached to an IAM Group. Add Marcus to this group with the following policy scope:
{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Allow",

      "Action": ["s3:GetObject", "s3:ListBucket", "s3:PutObject"],

      "Resource": [

        "arn:aws:s3:::riverside-inventory",

        "arn:aws:s3:::riverside-inventory/*"

      ]

    }

## Escalation Notes
The access gap analysis and recommended policy scope have been documented for escalation. Per HarborTech operational governance, all tier-1 analysts and interns do not make production IAM modifications. An authorized tier-2 or tier-3 analyst must review and deploy the proposed policy and group creation to prevent unauthorized policy drift and ensure account compliance.

## Lessons Learned
Week 2 taught me the difference between Authentication and authorization: authentication proves identity with a successful console login but does not grant permission to execute API actions. I learned that least-privilege standard permissions must be restricted to specific API actions and have exact resource names rather than broad managed policies. The CLI Investigation taught me to use tools like aws sts get-caller-identity and aws iam list-attached-role-policies to gather evidence during troubleshooting and verify accounts or role-managed policy details. 

## Professional Vocabulary
Define the important Week 2 terms in your own words.
Include terms such as:
- Authentication: The process to verify an accounts identity of a user when it comes to signing into AWS.
- Authorization: Actions that individuals can have permissions to manage, delete, or create resources.
- IAM: Used to securely control access to AWS resources.
- Policy: JSON document that formally defines permissions granted or denied to an identity or resource.
- Least Privilege: Grants only the minimum permissions necessary to perform an approved job function.
- AccessDenied: An error response returned when an identity attempts an action without explicit allow permissions.
- AWS CLI: A Command-line tool used to programmatically interact with AWS services.
- CloudShell: A browser-based shell environment provided in the AWS Console for executing CLI commands.
- Caller Identity: The AWS account, user ARN, and ID.
- Resource Scope: Specific target to which an IAM policy statement applies.
