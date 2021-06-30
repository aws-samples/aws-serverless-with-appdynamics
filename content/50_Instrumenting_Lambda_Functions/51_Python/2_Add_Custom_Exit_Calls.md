+++
title = "Add Custom Exit Calls"
chapter = false
weight = 2
+++

## Add Custom Exit Calls

Before proceeding with this part of the exercise, make sure that you have completed the section for [Adding Instrumentation to your Python Lambda function]({{< ref "./1_Add_Instrumentation.md" >}}).

During the initial setup phase, the setup script created an environment variable to make the configuration and deployment of instrumentation easier. Before we start configuring our Lambda functions, let's make sure the environment variable is still available. To check, run the following in a terminal:

``` bash
echo $AWS_REGION
```

If the output is one of the AWS regions, then you are good to go! Otherwise, execute the following command to save the current region as an environment variable:

``` bash
export AWS_REGION=$(aws configure get region)
```

In this section, we are going add custom code to show an exit call to an S3 bucket in the AppDynamics flowmap. Calls to other services that are not auto-detected by the Python tracer can be done in the same manner.

In the Cloud9 workspace, expand the *appd_aws_lambda_lab* folder, then expand the *python* subfolder. We will be making changes to **handler.py** and **handler2.py**, so go ahead and open those files.

![image](/images/instrumenting_lambda_functions/python/Python_Open_Files.png)

The first file we will be changing is **handler.py**. Within this file, we need to import the AppDynamics tracer into the code. On line 1, you will see a to-do comment to import the tracer. Add the following code on the following line:

``` python
import appdynamics
```

{{% notice note %}}
Because we are importing the AppDynamics tracer into the Lambda functions as a Lambda layer, we do not have to include the tracer directly in the deployed Lambda function code payload.
{{% /notice %}}

Within **handler.py**, locate the following code block (this should be starting around line 47 or line 48). This block of code uploads a file to an AWS S3 bucket.

``` python
        #TODO: Add in S3 exit call
        try:
            s3_client = boto3.client('s3')
            s3_client.put_object(Body=profile, Bucket=os.environ["CANDIDATE_S3_BUCKET"], Key=key)

            body = {
                "message": "Uploaded successfully.",
                "file" : key
            }

            retval = {
                "statusCode": 201,
                "body": json.dumps(body)
            }
        except Exception as e:

            body = {
                "message": "Could not upload resume."
            }

            retval = {
                "statusCode": 500,
                "body": json.dumps(body)
            }

       #TODO: End exit call
```

In the AppDynamics flowmap, this is represented as a connection to a third-party HTTP endpoint. But by adding some code to the function, we can have the connection represented as a connection to AWS S3. We will create a wrapped resource to create the exit call. Below the comment to add in the S3 exit call, add the below code.

``` python
with appdynamics.ExitCallContextManager(exit_point_type="CUSTOM", exit_point_sub_type="Amazon S3", identifying_properties={"BUCKET NAME" : os.environ["CANDIDATE_S3_BUCKET"]}) as ec:
```

Next, indent the remainder of the code block so that it falls under the exit call context manager. The function definition should look like the following:

``` python
def lambda_function(event, context):
    retval = {}

    if event['path'] == "/resume/random":
        lambda_client = boto3.client('lambda')
        response = lambda_client.invoke(
            FunctionName = context.function_name.replace("lambda-1", "lambda-2"),
            InvocationType = 'RequestResponse'
        )

        responsePayload = response['Payload'].read().decode('utf-8')

        if responsePayload is None:

            retval = {
                "statusCode" : 404,
                "body" : None
            }
        elif randint(1, 100) == 74:

            retval = {
                "statusCode" : 500,
                "body" : "Unknown Error"
            }
        else:
            retval = {
                "statusCode" : 200,
                "body" : responsePayload
            }
    else:
        faker = Faker()

        profile = json.dumps(faker.profile(["job", "company", "ssn", "residence", "username", "name", "mail"]))
        key = uuid.uuid4().hex + ".json"

        #TODO: Add in S3 exit call
        with appdynamics.ExitCallContextManager(exit_point_type="CUSTOM", exit_point_sub_type="Amazon S3", identifying_properties={"BUCKET NAME" : os.environ["CANDIDATE_S3_BUCKET"]}) as ec:
            try:
                s3_client = boto3.client('s3')
                s3_client.put_object(Body=profile, Bucket=os.environ["CANDIDATE_S3_BUCKET"], Key=key)

                body = {
                    "message": "Uploaded successfully.",
                    "file" : key
                }

                retval = {
                    "statusCode": 201,
                    "body": json.dumps(body)
                }
            except Exception as e:

                body = {
                    "message": "Could not upload resume."
                }

                retval = {
                    "statusCode": 500,
                    "body": json.dumps(body)
                }

       #TODO: End exit call

    return retval
```

The instrumented **handler.py** can be found [here](https://github.com/Appdynamics/appd_aws_lambda_lab/blob/instrumented/python/handler.py).

Save **handler.py**. We will now make similar changes to **handler2.py**, so go ahead and open that file in your Cloud9 workspace.

{{% notice note %}}
The tracer automatically reports errors that occur within an exit call. If you have a scenario where a resulting execution is considered an error, you can manually report custom exit calls. See [the documentation](https://docs.appdynamics.com/21.6/en/application-monitoring/install-app-server-agents/serverless-apm-for-aws-lambda/python-serverless-tracer/python-serverless-tracer-api) for more information.
{{% /notice %}}

On line 1 in **handler2.py**, you will see a to-do comment to import the tracer. Add the following code on the following line:

``` python
import appdynamics
```

There are 2 commented sections within this file where we will be adding custom exit calls. For each of these sections, add the below code snippet to each section immediately below each respective comment then immediately indent the next line to encapsulate it within the exit call manager variable. The code should look like this for each section:

``` python
    #TODO: Add in S3 exit call
    with appdynamics.ExitCallContextManager(exit_point_type="CUSTOM", exit_point_sub_type="Amazon S3", identifying_properties={"BUCKET NAME" : os.environ["CANDIDATE_S3_BUCKET"]}) as ec:
        objs = s3_client.list_objects_v2(Bucket = os.environ["CANDIDATE_S3_BUCKET"])['Contents']
```

``` python
    #TODO: Add in S3 exit call
    with appdynamics.ExitCallContextManager(exit_point_type="CUSTOM", exit_point_sub_type="Amazon S3", identifying_properties={"BUCKET NAME" : os.environ["CANDIDATE_S3_BUCKET"]}) as ec:
        obj = s3_client.get_object(Bucket = os.environ["CANDIDATE_S3_BUCKET"], Key = obj_key)
```

{{% notice note %}}
Information on the variables to pass when starting the exit call can be fouond in the [product documentation for the Python tracer](https://docs.appdynamics.com/21.6/en/application-monitoring/install-app-server-agents/serverless-apm-for-aws-lambda/python-serverless-tracer/python-serverless-tracer-api).
{{% /notice %}}

An example of the instrumented **handler2.py** can be found [here](https://github.com/Appdynamics/appd_aws_lambda_lab/blob/instrumented/python/handler2.py).

Save **handler2.py**. We have added custom exit calls to our Lambda functions, and now we will deploy our updates. Switch back to the terminal window in the Cloud9 workspace. Change to the *python* directory under the repository, then execute a deploy of the Python Lambda function as shown in the snippet below.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/python
serverless deploy -r $AWS_REGION
```

Allow about 5 minutes for the updated functions to be deployed and start being executed. Then switch back to the window where AppDynamics is running and click on **Application Dashboard** in the application menu on the left-hand side of the screen. You will now see the two Python Lambda functions connected to an AWS S3 instance on the flowmap.

![image](/images/instrumenting_lambda_functions/python/Python_S3_Exit_Call.png)

{{% notice note %}}
The custom exit call to the S3 bucket that we just added and the HTTP endpoint that was auto-detected are the same call -- we're just representing it with a different icon for ease of understanding.
{{% /notice %}}

You did it!! Not only did you successfully add instrumentation to an AWS Lambda function written in Python, but you also added in custom exit calls! Using AppDynamics to monitor your Python Lambda functions provides visibility into not only the performance of the functions, but also a holistic view of AWS services being consumed by the Lambda function and the application as a whole. That is a cause for celebration!

![image](https://media.giphy.com/media/JdCz7YXOZAURq/source.gif)

Now that you have instrumented a Python Lambda function, it's time to move on to instrumenting [NodeJS]({{< ref "../52_NodeJS/" >}}) or [Java]({{< ref "../53_Java/" >}}) Lambda functions. If you have completed your instrumentation, let's [clean up]({{< ref "../../60_Teardown/" >}}) any resources we may have used.
