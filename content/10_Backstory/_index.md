+++
title = "Backstory"
chapter = false
weight = 10
+++

![image](/images/ad_financial_logo_lrg.png)

AD Financial is a financial products and services company. They provide digital retail banking capabilities across the globe.

They currently have a home-grown legacy application that handles internal job postings and candidate selection. Due to explosive growth in hiring, some components of this application were recently refactored into a serverless architecture to use Amazon Lambda, Amazon API Gateway, Amazon S3, and Amazon DynamoDB. [Amazon Lambda](https://aws.amazon.com/lambda/) allows execution of code without needing to provision or manage server infrastructure. This provides flexibility in being able to rapidly deploy and update services while keeping costs down by paying only for the compute time used.

Olivia is the Vice President of Applications, Observability, and has tasked her organization with ensuring that all facets of applications within the organization are monitored. AD Financial is using AppDynamics to monitor the application.  But with the migration of some components to AWS Lambda,  monitoring on the AWS Lambda functions has not been setup yet. Without monitoring for Lambda functions, Olivia will not have observability into all components of the application.

You will be working with Miguel, lead monitoring engineer at AD-Financial, to implement monitoring.
