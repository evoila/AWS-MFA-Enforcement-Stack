# AWS MFA Enforcement Stack

This README provides a detailed overview of the AWS MFA Enforcement Stack and its resources, which automate the requirement for IAM users to have Multi-Factor Authentication (MFA) enabled if they possess console access.

### Description

AWS MFA Enforcement Stack is an AWS CloudFormation template that can be used to enforce Multi-Factor Authentication (MFA) for IAM users with console access. The CloudFormation stack sets up an environment where IAM users without MFA are restricted to managing their credentials until MFA is enabled for their accounts.

### Resources Created by This Stack

The stack creates several resources to support this functionality:

- `EnforceMFAPolicy`: An IAM managed policy that only allows users to manage their credentials and MFA devices.
- `Group`: An IAM group to which non-compliant users will be added. This group is attached to the `EnforceMFAPolicy`.
- `IamUserMfaEnabled`: A Config rule that checks if IAM users have MFA enabled.
- `RemediationConfiguration`: A Config remediation that triggers an SNS notification when the `IamUserMfaEnabled` rule finds a non-compliant user.
- `LambdaLogGroup`: A Log Group for logging outputs from the Lambda function.
- `LambdaExecutionRole`: An IAM role with permissions for the Lambda function to manage users and log events.
- `FunctionInvokePermission`: Grants the SNS topic permission to invoke the Lambda function.
- `LambdaFunction`: A Lambda function that adds non-compliant IAM users to the restricted IAM group.
- `SNSTopic`: An SNS topic for publishing notifications about non-compliant IAM users.
- `SNSSubscription`: A subscription that links the `SNSTopic` to the `LambdaFunction` for automatic triggering.
- `SNSPublisherRole`: An IAM role that allows SSM to publish to the `SNSTopic`.
- `SNSTopicPolicy`: Policy defining who can publish to the `SNSTopic`.

### Deployment Instructions

1. Log in to your AWS Management Console.
2. Navigate to the AWS CloudFormation service.
3. Choose to create a new stack and either upload the provided CloudFormation template file or input the Amazon S3 URL where the template is hosted.
4. Follow the prompts in the CloudFormation Wizard to specify stack details and configure stack options.
5. Review the stack details, acknowledging any capabilities that require acknowledgement (e.g., IAM resource creation).
6. Click "Create stack" to deploy the stack to your AWS account.

### How the Stack Works

After deployment:

1. AWS Config will periodically check the compliance status of IAM users against the `IamUserMfaEnabled` rule requirement.
2. Upon detecting non-compliant users, an SNS notification is sent to the `SNSTopic`.
3. The `SNSSubscription` will trigger the `LambdaFunction` when a message is published to the `SNSTopic`.
4. The `LambdaFunction` runs, adding the identified non-compliant user to the `Group` with the attached `EnforceMFAPolicy` that restricts actions to MFA management.
5. The user will be compelled to set up MFA before they can perform any other actions in the AWS Management Console.

### Prerequisites

- AWS account with sufficient permissions to create the resources listed above.
- Appropriate IAM permissions for the creation and execution of CloudFormation stacks, AWS Config rules, Lambda functions, IAM Managed Policies, and SNS topics.
- Knowledge of CloudFormation and the AWS environment for potential stack customization.

### Customization

If additional actions are required for compliance or if there are specific actions that should still be allowed without MFA, modify the `EnforceMFAPolicy` within the template. Ensure to test any changes in a controlled environment before deploying to production.

### Security Considerations

This stack implements the least privilege principle by only allowing IAM users to manage their credentials and MFA devices until they comply with the MFA policy. Audit your environment regularly to ensure that security policies are being enforced as intended.

### Support

This stack is provided as-is, with no warranties. For any issues or feature requests, please use GitHub issues on the repository page where this CloudFormation template is hosted.
