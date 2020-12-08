+++
title = "Add Configuration"
chapter = false
weight = 1
+++

## Configuring Lambda functions for Instrumentation

During the initial setup phase, the setup script created an environment variable to make the configuration and deployment of instrumentation easier. Before we start configuring our Lambda functions, let's make sure the environment variable is still available. To check, run the following in a terminal:

``` bash
echo $AWS_REGION
```

If the output is one of the AWS regions, then you are good to go! Otherwise, execute the following command to save the current region as an environment variable:

``` bash
export AWS_REGION=$(aws configure get region)
```

In the Cloud9 workspace, expand the *appd_aws_lambda_lab* folder, then expand the *java* subfolder. Double-click on the file **serverless.yml** to open.

![image](/images/instrumenting_lambda_functions/java/Java_Serverless_YAML.png)

This file is used by the Serverless framework to provision the AWS Lambda function and all of the other AWS resources needed by the Lambda function. This file contains multiple sections, but the important ones are highlighted below:

- **provider**: this contains information specific to the provider to where this code will be deployed (in this case, AWS). This includes environment variables, access permissions, and other relevant information. Within this section, we will be updating the entries underneath the **environment** sub-section.
- **resources**: this section contains other AWS resources that are provisioned when this is deployed for the first time. Since we have already deployed this during the initial setup, we will not be making updates to this section.
- **functions**: this section contains information about the AWS Lambda functions being deployed (including resource allocation, timeout length, function-specific environment variables, etc.). Multiple functions can be within this section. We will be updating each of the functions within this section to add our AWS Lambda Extension.

Next, open up the file named **lambda-environment-vars.txt** located in the *appd_aws_lambda_lab* folder. This file contains values for all of the environment variables that will be updated within **serverless.yml**.

![image](/images/instrumenting_lambda_functions/python/Lambda_Environment_Vars.png)

With both files open, take the values for each of the variables listed in **lambda-environment-vars.txt** and set the corresponding variables under the **provder** section in **serverless.yml** to the appropriate value. Next, set the value of the **APPDYNAMICS_SERVERLESS_API_ENDPOINT** environment variable to `https://pdx-sls-agent-api.saas.appdynamics.com`.

{{% notice note %}}
The tracer sends metrics collected from each Lambda function invocation to the API endpoint specified in the **APPDYNAMICS_SERVERLESS_API_ENDPOINT** environment variable. In the above example, the API endpoint is located in the US. There are API endpoints available in Europe (`https://fra-sls-agent-api.saas.appdynamics.com`) and Asia Pacific (`https://syd-sls-agent-api.saas.appdynamics.com`).
{{% /notice %}}

Save your changes. [In the next section]({{< ref "./2_Automatic_Instrumentation.md" >}}), we will be adding code to our Java Lambda function to add automatic instrumentation to one of our Lambda functions.
