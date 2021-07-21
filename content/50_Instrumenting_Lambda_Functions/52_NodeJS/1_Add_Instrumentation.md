+++
title = "Add Instrumentation"
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

In the Cloud9 workspace, expand the *appd_aws_lambda_lab* folder, then expand the *node* subfolder. Double-click on the file **serverless.yml** to open.

![image](/images/instrumenting_lambda_functions/node/Node_Serverless_YAML.png)

This file is used by the Serverless framework to provision the AWS Lambda function and all of the other AWS resources needed by the Lambda function. This file contains multiple sections, but the important ones are highlighted below:

- **provider**: this contains information specific to the provider to where this code will be deployed (in this case, AWS). This includes environment variables, access permissions, and other relevant information. Within this section, we will be updating the entries underneath the **environment** sub-section.
- **resources**: this section contains other AWS resources that are provisioned when this is deployed for the first time. Since we have already deployed this during the initial setup, we will not be making updates to this section.
- **functions**: this section contains information about the AWS Lambda functions being deployed (including resource allocation, timeout length, function-specific environment variables, etc.). Multiple functions can be within this section. We will be updating each of the functions within this section to add our AWS Lambda Extension.

Next, open up the file named **lambda-environment-vars.txt** located in the *appd_aws_lambda_lab* folder. This file contains values for all of the environment variables that will be updated within **serverless.yml**.

![image](/images/instrumenting_lambda_functions/python/Lambda_Environment_Vars.png)

With both files open, take the values for each of the variables listed in **lambda-environment-vars.txt** and set the corresponding variables under the **provder** section in **serverless.yml** to the appropriate value. Next, set the value of the **APPDYNAMICS_SERVERLESS_API_ENDPOINT** environment variable to `https://pdx-sls-agent-api.saas.appdynamics.com`.

{{% notice note %}}
The tracer sends metrics collected from each Lambda function invocation to the API endpoint specified in the **APPDYNAMICS_SERVERLESS_API_ENDPOINT** environment variable. In the above example, the API endpoint is located in the US. There are API endpoints available in Europe (https://fra-sls-agent-api.saas.appdynamics.com) and Asia Pacific (https://syd-sls-agent-api.saas.appdynamics.com).
{{% /notice %}}

Finally, create a new variable called **AWS_LAMBDA_EXEC_WRAPPER** and set its value to `/opt/appdynamics-extension-script`. The changes should look similar to the screenshot below.

![image](/images/instrumenting_lambda_functions/node/Node_Serverless_with_Vars.png)

Save your changes, and now we will add the AppDynamics AWS Lambda Extension to each of the Lambda functions. Locate the **functions** section within **serverless.yml**. Within that section will be 2 functions: *lambda-1* and *lambda-2*. To each of these functions, add the following:

``` yaml
    layers:
      - arn:aws:lambda:${opt:region, self:provider.region}:716333212585:layer:appdynamics-lambda-extension:10
```

The AppDynamics AWS Lambda Extension is published as a Lambda layer, and it takes advantage of the AWS Lambda Extensions API. In this snippet, we're telling our deployment to base the region of our Lambda layer on the region where our Lambdas are deployed. The resulting change should look like the screenshot below.

![image](/images/instrumenting_lambda_functions/node/Serverless_Lambda_Layers.png)

{{% notice note %}}
If you have already instrumented the Python Lambda function, you'll notice that these functions are missing the **APPDYNAMICS_TIER_NAME** environment variable that was present in the Python Lambda functions. This is an optional environment variable -- by default it will be set to a combination of the name of the Lambda function plus the version.
{{% /notice %}}

{{% notice note %}}
The AppDynamics AWS Lambda Extension is only available in certain AWS regions. For the latest set of regions where it is available, please see the [AppDynamics documentation](https://docs.appdynamics.com/21.6/en/application-monitoring/install-app-server-agents/serverless-apm-for-aws-lambda/use-the-appdynamics-aws-lambda-extension-to-instrument-serverless-apm-at-runtime). Also, the region from where you use the AppDynamics AWS Lambda Extension must match the region where your Lambda functions are running.
{{% /notice %}}

Save your changes. Switch back to the terminal window in the Cloud9 workspace. Change to the *node* directory under the repository, then execute a deploy of the NodeJS Lambda function by executing the commands in the snippet below.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/node
npm install
serverless deploy -r $AWS_REGION
```

This will first install the dependencies needed by the NodeJS code (in the Python Lambda function, the dependency installation is done via a Serverless plugin). Next, we trigger a deployment of Node Lambda functions with the new environment values and AppDynamics AWS Lambda Extension. After deployment has completed, allow five minutes for all the instances of the previous versions of our Lambda functions to complete exection and for the new versions to start being executed. Then switch back to the window where AppDynamics is running and click on **Tiers &amp; Nodes** in the application menu on the left-hand side of the screen. You should see our two Lambda functions listed in this screen.

![image](/images/instrumenting_lambda_functions/node/Node_Lambda_Tiers.png)

Next click on **Application Dashboard** in the application menu on the left-hand side of the screen. You will now see the two NodeJS Lambda functions appearing on the flowmap. You'll be able to distinguish these from the Pythoon Lambda functions (if you've instrumented them previously) by the NodeJS icon in the bottom left of the tier.

![image](/images/instrumenting_lambda_functions/node/Node_Lambda_Flowmap.png)

{{% notice note %}}
For now, you will still also see the third-party HTTP endpoint that represents the AWS API Gateway that hooks into our first NodeJS Lambda function. This will continue to appear on there until the duration of the selected timeframe passes.
{{% /notice %}}

Congratulations, you've successfully instrumented your first AWS Lambda function written in NodeJS using AppDynamics!

![image](https://media.giphy.com/media/ZUomWFktUWpFu/source.gif)

Thanks to your work, this application not only has visibility into the NodeJS Lambda function being called from our candidate-services application component, but we also now have visibility into other downstream Lambda functions as well!

Our NodeJS Lambda functions are making calls to an AWS DynamoDB instance. However, as previously mentioned, those calls are being detected as calls to a third-party HTTP endpoint. [In the next section]({{< ref "./2_Add_Custom_Exit_Calls.md" >}}), we will be adding code to our NodeJS Lambda function to create a custom exit call for DynamoDB.
