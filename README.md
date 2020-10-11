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


## Set up and Test

1. **Datadog - Initial Setup** 
 * From https://app.datadoghq.com/account/settings#api create an API Key
2. ** AWS Setup - AWS Control Tower Master account**
 * Launch the aws-datadog-controltower-v1.yml template in the AWS Control Tower Master account
 	-  Enter the API Key above. Accept all defaults
 	-  Ensure that a AWS CloudFormation StackSet is successfully created for the Datadog forwarder template
 	-  Ensure that a Amazon CloudWatch Events rule is successfully created with an AWS Lambda target to handle Control Tower Lifecycle events
  * Launch the aws-datadog-ct-cloudtrailcwlogs-v1 in the AWS Control Tower Master Account. Enter your email as input.
3. **Test - Create a Lifecycle Event - Add a managed account** 
 * From the AWS Control Tower Master Account:
    - Use Account Factory or quick provision or Service Catalog to create a  new managed account in the AWS Control Tower Organization OR
    - Use Service Catalog (AccountFactory Product) to update an existing managed account - for e.g. change the OU of an existing managed account
 	- This can take up to 30 mins for the account to be sucessfully created and the AWS Control Tower Lifecycle Event to trigger
 	- Login to the AWS Control Tower managed account - 
 		- Validate that an AWS CloudFormation stack instance has been provisioned that launches the Datadog Forwarder template in the managed account. 
 		- Validate that a Datadog Integration Role (DatadogIntegrationRole IAM role) has been created in the managed account.  This is a cross account role where the trusted account ID - 464622532012 corresponds to the Datadog control plane.
4. **Datadog - Complete Setup** 
 * From https://app.datadoghq.com/account/settings#integrations/amazon-web-services click on 'Add Account'->'Role Delegation'->Manual. Add the Account ID of the AWS Control Tower managed account and Role Name(DatadogIntegrationRole).Copy the generated External ID. 
 	- Go back to the AWS Control Tower managed account and update the External ID in the DatadogIntegrationRole with the External ID that was generated in the Datadog console
 	- Click on "Update Configuration" in Datadog console to complete the setup
5. **Test - Create a Control Tower Lifecyle Event** 
 * From the AWS Control Tower Master Account:
    - Create a Control Tower Lifecycle event that is not related to creating/updating managed account. For e.g. enable a new detective non mandatory Guardrail on an OU.
    - For quick validation - check that an email has been received which describes the event as logged in CloudTrail
