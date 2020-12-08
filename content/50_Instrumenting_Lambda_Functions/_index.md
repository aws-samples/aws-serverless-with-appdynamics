+++
title = "Instrumenting Lambda Functions"
chapter = false
weight = 50
+++

## Instrumenting Lambda Functions

In our sandbox environment, the Lambda functions being utilized were written by different teams with different areas of expertise. As a result, the Lambda functions were written in three different languages: Python, NodeJS, and Java. Deployment of all Lambda functions is standardized on the Serverless framework.

AppDynamics supports monitoring AWS Lambda functions written in Python (version 3.6 and greater), NodeJS (version 10.x and greater), and Java (version 1.8 and greater). Instrumentation is accomplished through using a tracer module that captures relevant data. The tracer module does not depend on any other AWS services. The tracer captures performance information for each invocation of the Lambda function and transmits it asynchronously to AppDynamics.

For functions written in Python and NodeJS, AppDynamics has integrated with [AWS Lambda Extensions](https://aws.amazon.com/blogs/compute/introducing-aws-lambda-extensions-in-preview/) feature to create a Lambda Layer to automatically handle instrumentation.
![image](/images/instrumenting_lambda_functions/Node_Python_Lambda_Layer.png)

For Java-based Lambda functions, the tracer is embedded as an SDK directly within the function.
![image](/images/instrumenting_lambda_functions/How_It_Works.png)

Now check out the following sections to learn how to instrument Lambda functions with AppDynamics.

- [Python]({{< ref "51_Python/" >}})
- [NodeJS]({{< ref "52_NodeJS/" >}})
- [Java]({{< ref "53_Java/" >}})

{{% notice note %}}
You may complete the sections for any combination of languages. However, it is highly recommended to complete the following sections: **Instrumenting Lambda Functions > Python** and **Instrumenting Python Lambda Functions > NodeJS**
{{% /notice %}}
