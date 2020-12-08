+++
title = "AWS Account"
chapter = false
weight = 2
+++

{{% notice warning %}}
You are responsible for the cost of the AWS services used while running this workshop in your AWS account.
{{% /notice %}}

## Minimum User Access

For this lab, you will need to be logged in as a user that has the following priviliges:

- Full access to the following AWS services: Lambda, DynamoDB, API Gateway, CloudFormation, CloudWatch, S3, Cloud9, Logs
- Access to the following capabilities within EC2:
  - Describe instances / subnets / VPCs
  - Describe volumes / volume attributes / volume status
  - Modify volumes / volume attributes
- Access to the following capabilities within IAM:
  - Create / get / put / update / delete / pass roles
  - Create / delete service-linked roles
  - Get / create / delete policies
  - Attach / put / delete role policies

If your account does not have these capabilities (or if you prefer to walk through this exercise with a different user account), see the **Create an Account** section below. Otherwise you are ready to start [the workshop setup]({{< ref "../30_Workshop_Setup/" >}}).

## Create an account

Below are links to CloudFormation templates to create a user account with the appropriate permissions to run this lab. Click the one that is appropriate to your region to launch the CloudFormation template. Make sure to only click one of the links. If you don't have permissions to create users and policies via CloudFormation templates, ask someone in your organization with access to do this for you.

| Launch Template Options | Region |
|:-----------------------:|--------|
|  [![Deploy to AWS](/images/getting_started/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?stackName=appd-lambda-lab&templateURL=https://appd-lambda-lab-workshop.s3.amazonaws.com/appd-lambda-lab-cf-template.yaml)| N. Virginia (us-east-1)|
|  [![Deploy to AWS](/images/getting_started/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/create/review?stackName=appd-lambda-lab&templateURL=https://appd-lambda-lab-workshop.s3.amazonaws.com/appd-lambda-lab-cf-template.yaml)| Ohio (us-east-2)|
|  [![Deploy to AWS](/images/getting_started/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/create/review?stackName=appd-lambda-lab&templateURL=https://appd-lambda-lab-workshop.s3.amazonaws.com/appd-lambda-lab-cf-template.yaml)| N. California (us-west-1)|
|  [![Deploy to AWS](/images/getting_started/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?stackName=appd-lambda-lab&templateURL=https://appd-lambda-lab-workshop.s3.amazonaws.com/appd-lambda-lab-cf-template.yaml)| Oregon (us-west-2)|
|  [![Deploy to AWS](/images/getting_started/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/create/review?stackName=appd-lambda-lab&templateURL=https://appd-lambda-lab-workshop.s3.amazonaws.com/appd-lambda-lab-cf-template.yaml)| Ireland (eu-west-1)|
|  [![Deploy to AWS](/images/getting_started/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/create/review?stackName=appd-lambda-lab&templateURL=https://appd-lambda-lab-workshop.s3.amazonaws.com/appd-lambda-lab-cf-template.yaml)| Singapore (ap-southeast-1)|

Once you have created the user for this lab, let's start [the workshop setup]({{< ref "../30_Workshop_Setup/" >}}) to create our Cloud9 workspace and get started.
