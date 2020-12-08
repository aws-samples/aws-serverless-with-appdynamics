+++
title = "Java"
chapter = false
weight = 53
+++

## Objective

In this section of the workshop, we are going to add AppDynamics instrumentation to a set of AWS Lambda functions written in Java. We will walk through the steps of adding the AppDynamics Tracer SDK to the Java application. We will also configure the deployment with the needed environment variables and add the necessary code to setup instrumentation and create exit calls to another Lambda function.

## Introduction to AppDynamics Serverless APM for AWS Lambda -- Java
Appdynamics AWS Lambda Extension for Java application is not available as of this lab (December 2020). 

In this exercise, we will utilize the Java tracer SDK to demonstrate how customers can instrument Amazon Lambda functions in Java. 

Using the AppDynamics to provide observability into Lambda functions gives application owners, DevOps engineers, and IT Operations engineers the visibility needed to understand all of the components of the application, serverless or traditional, and be able to proactively address issues that impact business revenue.

## Exercise

This exercise is broken down into three sections:

- [Configuring Lambda function deployment]({{< ref "./1_Add_Configuration.md" >}})
- [Adding custom code for automatic instrumentation]({{< ref "./2_Automatic_Instrumentation.md" >}})
- [Adding custom code for manual instrumentation]({{< ref "./3_Manual_Instrumentation.md" >}})
