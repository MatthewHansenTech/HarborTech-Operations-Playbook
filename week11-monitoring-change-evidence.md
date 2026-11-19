# Week 11: Monitoring and Change Evidence

## HarborTech Ticket Summary

Ticket ID: TKT-2026-0011

HarborTech was asked to investigate an unexpected configuration change within the AWS environment. The objective was to reconstruct the incident using monitoring and audit evidence rather than memory. The investigation focused on identifying what changed, when the change occurred, who initiated it, and what evidence supported the conclusion.

## Client Impact
Unauthorized or undocumented configuration changes can increase operational and security risk. Changes made without approval or documentation make troubleshooting more difficult and can expose cloud resources to unintended access. HarborTech was asked to determine what happened and recommend appropriate corrective actions based on available evidence.

## AWS Services Involved
* Amazon CloudWatch
* CloudWatch Metrics
* CloudWatch Alarms
* AWS CloudTrail
* Monitoring
* Audit Evidence
* Configuration Verification
* Incident Timeline Analysis

## Virtualization Connection

Virtualization allows infrastructure, monitoring, and management functions to operate through software-defined systems instead of physical hardware controls. In our investigation, CloudWatch provided performance and monitoring information, while CloudTrail supplied audit records showing management activity within the AWS environment. Together, these services allowed HarborTech to reconstruct the incident using objective system-generated evidence instead of relying on memory.

## Evidence Reviewed

The following evidence sources were reviewed during the investigation:

* CloudWatch alarm history
* CloudWatch alarm status
* CloudTrail management events
* Event timestamps
* User identity information
* API activity records
* Configuration-change evidence
* Incident timeline reconstruction

AWS Config was investigated as a potential evidence source; however, no configuration recorder was configured within our Learner Lab environment.

Amazon Athena was reviewed as a potential evidence source, but no investigation-related datasets were available for use during this incident.

## AWS Commands Used
### CloudTrail Audit Review
aws cloudtrail lookup-events --max-results 20

### CloudWatch Alarm History
aws cloudwatch describe-alarm-history \
--alarm-name week11-cpu-high

### Security Group Configuration Review
aws ec2 describe-security-groups \
--group-ids sg-000904af0cdf84e53

### Alarm Status Verification
aws cloudwatch describe-alarms \
--alarm-names week11-cpu-high

## AWS Evidence Collected
### CloudTrail Audit Evidence
Event Name:
AuthorizeSecurityGroupIngress

Event Time:
2026-11-17T21:17:30Z

Event Source:
ec2.amazonaws.com

User:
user4752033=Matthew_J._Hansen

Affected Resource:
sg-000904af0cdf84e53

Result:
Inbound TCP 8080 rule added

### CloudWatch Alarm History
2026-11-17T21:16:58 UTC
Alarm Created

2026-11-17T21:17:31 UTC
INSUFFICIENT_DATA → OK

2026-11-17T21:18:31 UTC
OK → ALARM

2026-11-17T21:56:31 UTC
ALARM → INSUFFICIENT_DATA

2026-11-17T21:58:31 UTC
INSUFFICIENT_DATA → OK

2026-11-18T23:01:54 UTC
INSUFFICIENT_DATA → ALARM

### CloudWatch Alarm Status
Alarm Name:
week11-cpu-high

Instance ID:
i-0eafd856c5daf1f97

Metric:
CPUUtilization

State:
ALARM

Threshold:
1.0

Observed Value:
6.739442235113652

State Updated:
2026-11-18T23:01:54 UTC

## Operational Analysis
The investigation began with monitoring evidence rather than configuration records. CloudWatch alarm history showed that the week11-cpu-high alarm was created at 21:16:58 UTC. The alarm later transitioned from INSUFFICIENT_DATA to OK and then from OK to ALARM, establishing a clear incident window and a reliable sequence of events.

Time (UTC)	Evidence Source	Event
21:16:58	CloudWatch	Alarm created
21:17:30	CloudTrail	AuthorizeSecurityGroupIngress event recorded
21:17:31	CloudWatch	Alarm transitioned from INSUFFICIENT_DATA to OK
21:18:31	CloudWatch	Alarm transitioned from OK to ALARM

CloudTrail audit records showed that an AuthorizeSecurityGroupIngress API action occurred at 21:17:30 UTC. The event was associated with the authenticated AWS session user. CloudWatch monitoring data provided timestamps immediately before and after the recorded management action. By correlating monitoring records, audit evidence, and event timestamps, HarborTech reconstructed the sequence of events and verified that a configuration change occurred during the incident window. The evidence clearly identifies the timing of the change and the AWS session associated with it. However, the available evidence does not explain the business purpose behind the modification.

### Supported Findings
Finding 1: CloudTrail recorded an AuthorizeSecurityGroupIngress event at 21:17:30 UTC.

Finding 2: CloudTrail identified the authenticated AWS session user4752033=Matthew_J._Hansen as the session associated with the modification.

Finding 3: CloudWatch alarm history documented monitoring activity and provided timestamps that supported reconstruction of the incident timeline.

Finding 4: CloudWatch confirmed that CPU utilization exceeded the configured threshold, causing the alarm to enter the ALARM state.

CloudWatch alarm history documented monitoring activity and recorded transitions from INSUFFICIENT_DATA to OK and later from OK to ALARM.

The available evidence indicates that the modification was performed through an authenticated AWS session rather than an automated system failure. The modification may have been performed for testing, troubleshooting, or temporary operational purposes; however, the available evidence does not confirm the reason for the change.

## Recommendation
HarborTech should review the identified configuration change against approved change-management records and determine whether the modification was authorized. If the change is not operationally required, the environment should be returned to its approved baseline configuration. Continue monitoring through CloudWatch and CloudTrail to identify future unauthorized or undocumented changes.

## Escalation Notes
Finding: A documented configuration change occurred during the investigation window, and CloudTrail identified the AWS session associated with the action.

Assumption: The available evidence confirms who made the change and when, but it does not establish the business justification for the modification.

Escalation: If no approved change documentation exists, escalate the incident to cloud operations or security personnel for review. Undocumented configuration changes should be validated against organizational change-management procedures.

## Lessons Learned
Week 11 demonstrated the importance of using evidence rather than memory during cloud investigations. CloudWatch established the incident timeline, while CloudTrail identified who performed the action and when. Individually, each source provided only part of the story. By correlating monitoring data, audit records, and timestamps, HarborTech reconstructed the incident and reached evidence-based conclusions.

## Professional Vocabulary
- Monitoring: Continuous observation of system performance and operational health.
- Metric: Measurable value collected from a system, such as CPU utilization.
- Log: Recorded event generated by a service, application, or system.
- Alarm: Monitoring object that evaluates metrics against predefined thresholds.
- CloudWatch: AWS monitoring service used to collect metrics and monitor operational activity.
- CloudTrail: AWS auditing service that records API activity and administrative actions.
- AWS Config: AWS service that records resource configurations and tracks configuration changes over time.
- Audit Trail: Record of actions performed within an environment.
- Configuration History: A record showing how resource settings change over time.
- Incident Timeline: A chronological reconstruction of events used during an investigation.
- Evidence Correlation: The process of combining information from multiple evidence sources to support conclusions and reconstruct incidents.
