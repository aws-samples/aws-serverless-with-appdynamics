+++
title = "Workshop Setup"
chapter = false
weight = 30
+++

## Workshop Setup

Miguel is a lead monitoring engineer at AD-Financial and will walk you through the steps to enable observability for the AWS Lambda components of the application. To allow you to become familiar with working with AppDynamics, the steps will be taken within a sandboxed development environment. Please review the steps below to ensure a smooth setup.

Below is an architecture diagram of the application we will be working with once instrumentation is complete.

![image](/images/architecture_diagram.png)

Below are the workshop setup steps you will need to complete before we start adding observability to our Lambda functions:

1. [Create your Cloud9 Workspace.]({{< ref "1_Create_Cloud9.md" >}})

    - The minimum instance type for the Cloud9 instance is **t3.medium**.
    - **Amazon Linux 2** should be the selected operating system.

2. [Install tooling, configure AppDynamics, and start the application.]({{< ref "2_Install_Tooling_Configure_AppD.md" >}})
