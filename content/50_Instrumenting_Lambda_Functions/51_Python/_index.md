+++
title = "Python"
chapter = false
weight = 51
+++

## Objective

In this section of the workshop, we are going to add AppDynamics instrumentation to a set of AWS Lambda functions written in Python. We will walk through the steps of configuring the Lambda functions to use the AppDynamics [AWS Lambda Extension](https://aws.amazon.com/blogs/compute/introducing-aws-lambda-extensions-in-preview/). Finally, we will add in custom code to show exit calls to AWS resources that are not automatically detected.

## Introduction to AppDynamics Serverless APM for AWS Lambda -- Python

Appdynamics released the [AppDynamics AWS Lambda Extension for Serverless APM](https://www.appdynamics.com/blog/product/enhancing-lambda-performance-monitoring/) in October 2020 as an enhancement to the serverless tracer for AWS Lambda, to make it easier to instrument Amazon Lambda functions in Python.

![image](/images/instrumenting_lambda_functions/Node_Python_Lambda_Layer.png)

Out of the box, the AppDynamics Python Lambda tracer automatically detects calls that are made to third-party HTTP services, AWS DynamoDB service endpoints, and other AWS Lambda functions. With some custom code added to the Lambda function, calls to other AWS serverless service endpoints (such as S3 etc.) can be reflected in the AppDynamics flowmap. As a result, using AppDynamics to provide observability into Lambda functions gives application owners, DevOps engineers, and IT Operations engineers the visibility needed to understand all of the components of the application, serverless or traditional, and be able to proactively address issues that impact business revenue.

## Exercise

This exercise is broken down into two sections:

- [Configuring Lambda functions for automatic instrumentation]({{< ref "./1_Add_Instrumentation.md" >}})
- [Adding custom code for exit calls to S3]({{< ref "./2_Add_Custom_Exit_Calls.md" >}})
