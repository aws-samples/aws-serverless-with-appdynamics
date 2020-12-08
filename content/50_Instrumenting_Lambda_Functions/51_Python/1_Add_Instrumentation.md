+++
title = "Add Instrumentation"
chapter = false
weight = 1
+++

## Configuring Lambda functions for Instrumentation

Having visibility into the Lambda components of your application is crucial to understanding the entire landscape of your application. By default, downstream calls to Lambda functions from an instrumented upstream component appear as a third-party HTTP endpoint. However, your Lambda function may be calling other Lambda functions, working with S3 buckets, or incorporating any other number of AWS services. Without proper instrumentation of Lambda functions, there is no visibility into anything downstream from Lambda.

During the initial setup phase, the setup script created an environment variable to make the configuration and deployment of instrumentation easier. Before we start configuring our Lambda functions, let's make sure the environment variable is still available. To check, run the following in a terminal:

``` bash
echo $AWS_REGION
```

If the output is one of the AWS regions, then you are good to go! Otherwise, execute the following command to save the current region as an environment variable:

``` bash
export AWS_REGION=$(aws configure get region)
```

In the Cloud9 workspace, expand the *appd_aws_lambda_lab* folder, then expand the *python* subfolder. Double-click on the file **serverless.yml** to open.

![image](/images/instrumenting_lambda_functions/python/Python_Serverless_YAML.png)

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

Finally, create a new variable called **AWS_LAMBDA_EXEC_WRAPPER** and set its value to `/opt/appdynamics-extension-script`. The changes should look similar to the screenshot below.

![image](/images/instrumenting_lambda_functions/python/Python_Serverless_with_Vars.png)

{{% notice note %}}
The **AWS_LAMBDA_EXEC_WRAPPER** environment variable is only used for Python functions if the version of Python is 3.8. Versions 3.6 and 3.7 will use a different environment variable which is outlined in the [product documentation](https://docs.appdynamics.com/display/PRO45/Use+the+AppDynamics+AWS+Lambda+Extension+to+Instrument+Serverless+APM+at+Runtime).
{{% /notice %}}

Save your changes, and now we will add the AppDynamics AWS Lambda Extension to each of the Lambda functions. Locate the **functions** section within **serverless.yml**. Within that section will be 2 functions: *lambda-1* and *lambda-2*. To each of these functions, add the following:

``` yaml
    layers:
      - arn:aws:lambda:${opt:region, self:provider.region}:716333212585:layer:appdynamics-lambda-extension:9
```

The AppDynamics AWS Lambda Extension is published as a Lambda layer, and it takes advantage of the AWS Lambda Extensions API (currently under public preview). In this snippet, we're telling our deployment to base the region of our Lambda layer on the region where our Lambdas are deployed. The resulting change should look like the screenshot below.

![image](/images/instrumenting_lambda_functions/python/Serverless_Lambda_Layers.png)

{{% notice note %}}
The AppDynamics AWS Lambda Extension is only available in certain AWS regions. For the latest set of regions where it is available, please see the [AppDynamics documentation](https://docs.appdynamics.com/display/PRO45/Use+the+AppDynamics+AWS+Lambda+Extension+to+Instrument+Serverless+APM+at+Runtime). Also, the region from where you use the AppDynamics AWS Lambda Extension must match the region where your Lambda functions are running.
{{% /notice %}}

Save your changes. Switch back to the terminal window in the Cloud9 workspace. Change to the *python* directory under the repository, then execute a deploy of the Python Lambda function as shown in the snippet below.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/python
serverless deploy -r $AWS_REGION
```

This will trigger a deployment of Python Lambda functions with the new environment values and AppDynamics AWS Lambda Extension. After deployment has completed, allow five minutes for all the instances of the previous versions of our Lambda functions to complete exection and for the new versions to start being executed. Then switch back to the window where AppDynamics is running and click on **Tiers &amp; Nodes** in the application menu on the left-hand side of the screen. You should see our two Lambda functions listed in this screen.

![image](/images/instrumenting_lambda_functions/python/Python_Lambda_Tiers.png)

Next click on **Application Dashboard** in the application menu on the left-hand side of the screen. You will now see the two Python Lambda functions appearing on the flowmap.

![image](/images/instrumenting_lambda_functions/python/Python_Lambda_Flowmap.png)

Congratulations, you've successfully instrumented your first AWS Lambda function written in Python using AppDynamics!

![image](https://media.giphy.com/media/xTiTnEHBh7qapyuvwQ/giphy.gif)

Thanks to your work, this application not only has visibility into the Python Lambda function being called from our candidate-services application component, but we also now have visibility into other downstream Lambda functions as well! Using this approach from AppDynamics allows for easy deployment of observability within Lambda functions.

Our Python Lambda functions are making calls to AWS S3. However, as previously mentioned, those calls are being detected as calls to a third-party HTTP endpoint. [In the next section]({{< ref "./2_Add_Custom_Exit_Calls.md" >}}), we will be adding code to our Python Lambda function to create a custom exit call for S3.
