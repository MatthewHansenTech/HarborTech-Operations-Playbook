# Week 10: Storage, Recovery, and Data Retention

## HarborTech Ticket Summary
HarborTech received a request from Bright Path Community Services to review its AWS storage and recovery strategy. The client's most recent documented EC2 snapshot is 19 days old, even though the maximum acceptable data loss is 24 hours. The client also requires shared file access for three Linux servers, reliable storage for uploaded documents, and long-term retention of records that must be kept for seven years.

## Client Impact
If a server failure occurred, the client could lose a significant amount of data because backups are not being created frequently enough. The client also needs shared storage for multiple Linux servers and a reliable way to store uploaded documents. Records that are rarely accessed after 90 days must still remain available for long-term retention and compliance requirements.

## AWS Environment Verification

Command:

aws sts get-caller-identity

Observed Result:

Account ID: 956152099107

Command:

aws configure list

Observed Result:

Region: us-east-1

I verified that I was connected to the AWS Academy Learner Lab environment and confirmed that resources would be created in the us-east-1 Region.

## AWS Services Involved
The investigation involved several AWS storage services and storage management features. We reviewed these services to determine which storage solutions best supported the client's recovery, file sharing, document storage, and long-term retention requirements:

* Amazon EBS: Used to provide block storage for EC2 instances

* EBS Snapshots: Used to create backup and recovery points for EBS volumes

* Amazon EFS: Reviewed as a solution for shared file access across multiple Linux servers

* Amazon S3: Used to store uploaded documents as objects

* S3 Versioning: Used to protect files from accidental deletion or modification

* S3 Lifecycle Rules: Reviewed to automate data movement based on age and access patterns

* Glacier Storage Classes: Reviewed as a long-term archival storage solution

* Server-Side Encryption: Verified that uploaded S3 objects were protected using encryption

## Evidence Reviewed
### EBS Volume Investigation

Command:

aws ec2 describe-volumes --volume-ids vol-0a344ba1daddb24e0 --region us-east-1

Observed Result:

"VolumeId": "vol-0a344ba1daddb24e0"

"VolumeType": "gp2"

"Size": 1

"AvailabilityZone": "us-east-1a"

"State": "available"

The EBS volume was successfully created and was ready to be attached to an EC2 instance.

### EBS Snapshot Investigation

Command:

aws ec2 describe-snapshots --snapshot-ids snap-0f0e135473760fe5c

Observed Result:

"SnapshotId": "snap-0f0e135473760fe5c"

"State": "completed"

"Progress": "100%"

"Description": "IT4560 Week10 practice"

The snapshot was successfully completed and created a recovery point for the EBS volume.

S3 Object Storage Investigation

Command:

aws s3api head-object --bucket week10-matthew-hansen --key week10-sample.txt --region us-east-1

Observed Result:

"ContentLength": 23

"ContentType": "text/plain"

"ServerSideEncryption": "AES256"

The uploaded file was stored successfully and protected with server-side encryption.


### S3 Versioning Investigation

Command:

aws s3api get-bucket-versioning --bucket week10-matthew-hansen --region us-east-1

Observed Result:

"Status": "Enabled"

Versioning was successfully enabled for the bucket.

### Lifecycle Investigation

Command:

aws s3api get-bucket-lifecycle-configuration --bucket week10-matthew-hansen --region us-east-1

Observed Result:

### NoSuchLifecycleConfiguration

No lifecycle configuration existed for the bucket.

Proposed Lifecycle Rule:

```json
{
  "Rules": [
    {
      "ID": "ArchiveAfter90Days",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "records/"
      },
      "Transitions": [
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}

## Operational Analysis
My evidence shows that EBS provides storage for EC2 instances and supports snapshots for backup and recovery. The completed snapshot demonstrated that recovery points can be created successfully. However, a 19-day-old snapshot would not meet the client's current requirement to limit data loss to 24 hours. The S3 investigation confirmed that uploaded documents can be stored reliably using object storage. The uploaded file was stored successfully and protected with encryption, and versioning was also enabled, which helps protect files from accidental deletion or modification. The lifecycle investigation showed that no lifecycle policy currently exists. This means older records are not automatically moved into archival storage. Based on the evidence collected, multiple AWS storage services are needed to meet all client requirements.

## Recommendation
Based on the evidence I collected, I recommend using Amazon EBS with automated daily snapshots for EC2 storage and recovery. Daily snapshots would help keep backups within the client's 24-hour recovery requirement. For the three Linux servers that require shared access to files, I recommend Amazon EFS because it allows multiple servers to use the same file system at the same time. For uploaded documents, I also recommend Amazon S3 because it provides durable and scalable object storage. For records that are rarely accessed after 90 days but must be retained for seven years, I recommend implementing a Lifecycle Rule that transitions data into Glacier storage.

## Recovery and Retention Verification 
The snapshot investigation confirmed that AWS can create recovery points for EBS volumes. To meet the client's recovery objective, snapshots should be scheduled automatically at least once every 24 hours. The lifecycle investigation confirmed that no lifecycle policy currently exists. A lifecycle rule that moves records into Glacier storage after 90 days would help support the client's seven-year retention requirement.

## Resource Cleanup

Command:

aws ec2 delete-snapshot --snapshot-id snap-0f0e135473760fe5c

Observed Result:

Snapshot deleted successfully.

Command:

aws ec2 delete-volume --volume-id vol-0a344ba1daddb24e0

Observed Result:

Volume deleted successfully.

Command:

aws s3 rm s3://week10-matthew-hansen/week10-sample.txt

Observed Result:

Object deleted successfully.

Command:

aws s3 rb s3://week10-matthew-hansen

Observed Result:

BucketNotEmpty

The EBS volume, snapshot, and uploaded object were successfully removed. The bucket remained because versioned objects still existed within the bucket.

## Escalation Notes
Management should review and approve costs related to Amazon EFS, automated snapshots, and Glacier storage. The client's seven-year retention requirement should also be verified before applying lifecycle policies to production data.

## Lessons Learned
This week taught me that different AWS storage services are designed for different purposes. EBS is used for EC2 storage, EFS provides shared file access across multiple servers, and S3 stores uploaded documents and other files. I also learned how snapshots support recovery planning, how versioning protects files from accidental changes, and how lifecycle rules can automate long-term storage management.

## Professional Vocabulary
Block Storage: Storage that presents data as disk volumes to operating systems and applications.
File Storage: Storage that organizes information into files and folders that multiple systems can access.
Object Storage: Storage that stores data as objects along with metadata.
EBS Snapshot: A point-in-time backup of an Amazon EBS volume.
Amazon EFS: A managed file storage service that allows multiple systems to access the same file system.
S3 Versioning: A feature that stores multiple versions of files and helps recover deleted or modified data.
Lifecycle Rule: An automated policy that manages data based on age or storage requirements.
Archive: A Long-term storage for data that must be retained but is rarely accessed.
