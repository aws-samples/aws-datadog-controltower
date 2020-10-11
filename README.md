<p align="center">
</p>

# Use AWS Control Tower with Datadog for Multi account AWS Cloud Monitoring and Analytics

* AWS Control Tower Lifecyle Integration with Datadog - Allow new or updated AWS accounts in an AWS Control Tower based AWS Organization to be managed automatically by Datadog
* AWS Control Tower Master Account Integration with Datadog - Allow all AWS Control Tower Lifecycle Events (for e.g. events such as enable/disable guardrails on an OU) to be forwarded to Datadog


## How it Works

1. **Template: aws-datadog-controltower-v1.yml**:
 * This template provisions infrastructure in the Control Tower Master account that allows creation of Datadog stack instances in Control Tower managed accounts whenever a new Control Tower managed account is added
 * Creates a Datadog Stackset in the Control Tower Master Account 
 * Provisions a CloudWatchEvents Rule that is triggered based on a Control Tower Lifecycle Event
 * Provisions a Lifecyle Lambda as a target for the CloudWatch Events Rule
 	- The Lifecycle Lambda deploys a Datadog stack in the newly added Control Tower managed account--thus placing that account under Datadog management
 * The infrastructure provisioned by this template above allows for a Control Tower lifecycle event trigger specifically the CreateManagedAccount or UpdateManagedAccount events to:
	- Trigger the Lifecyle Lambda that creates Datadog stack instance in the managed account based on the Datadog stackset in the master account
 * All parameters that are needed for the Datadog Forwarder such as API Key and Secret are stored and retrieved from AWS Secrets Manager

2. **Template: aws-datadog-ct-cloudtrailcwlogs-v1**:
 * This template allows all Control Tower Lifecycle Events from the Control Tower Master Account to be forwarded to Datadog
 * Control Tower Lifecyle events are logged to CloudTrail in CloudWatch Logs provided during Control Tower set up
 * Provisions a CloudWatch Logs Metric Filter for the Control Tower CloudWatch Logs that filters based on Control Tower Lifecycle events
 * Provisions a CloudWatch Alarm that is triggered wheneven a metric condition is met 
 * Provisions SNS topic that notifies Datadog Forwarder Lambda as well has an optional email subscriber 
 

## Solution Design

![](images/arch-diagram1.png)


## Set up

1. **Validate** 
 * Login to the AWS Console from the Team Dashboard
 	-  Ensure that the CloudFormation template mod-* has fully deployed. Wait to proceed further until the template has successfully deployed
 	-  Check that the attack has been initiated in the AWS environment:
 		- CloudTrail (called 'GameDayTrail') has Log File Validation Disabled and CloudWatch Logs Monitoring Disabled
 		- KMS Customer Master Key (with description 'Test Key Rotation') has key rotation disabled
 		- Elastic IP has an unassociated Instance ID

2. **Detect and Remediate Attack** 
 * Login to the AWS Console from the Team Dashboard:
    - Turn on AWS Config
 	- Deploy the **'mg-gameday-module3-configremediations-v1'** CloudFormation Template
 	- Validate that AWS Config Managed Rules have been provisioned. Each Config Rule has an associated AWS Systems Manager Automation associated with it as a remediation
 	- After a few minutes check each of the misconfigured resources from the attack. Ensure that the attack has been thwarted and all resources are in a compliant state. Check the AWS Config Dashboard to see how Config has detected and then remediated the violations

3. **Earn Points** 
 * On the Team Dashboard:
    - Enter the name of the CloudFormation template that you deployed and click update
    - The Game will check the misconfigured resources have been updated and will award x points each for each remediated resource 