+++
title = "Add Custom Exit Calls"
chapter = false
weight = 2
+++

## Add Custom Exit Calls

Before proceeding with this part of the exercise, make sure that you have completed the section for [Adding Instrumentation to your NodeJS Lambda function]({{< ref "./1_Add_Instrumentation.md" >}}).

During the initial setup phase, the setup script created an environment variable to make the configuration and deployment of instrumentation easier. Before we start configuring our Lambda functions, let's make sure the environment variable is still available. To check, run the following in a terminal:

``` bash
echo $AWS_REGION
```

If the output is one of the AWS regions, then you are good to go! Otherwise, execute the following command to save the current region as an environment variable:

``` bash
export AWS_REGION=$(aws configure get region)
```

In this section, we are going add custom code to show an exit call to a DynamoDB instance in the AppDynamics flowmap. Calls to other services that are not auto-detected by the NodeJS tracer can be done in the same manner.

In the Cloud9 workspace, expand the *appd_aws_lambda_lab* folder, then expand the *node* subfolder. We will be making changes to **handler.js**, so go ahead and open that file.

![image](/images/instrumenting_lambda_functions/node/Node_Open_File.png)

Within this file, we need to import the AppDynamics tracer into the code. On line 1, you will see a to-do comment to import the tracer. Add the following code on the following line:

``` javascript
const tracer = require('appdynamics-lambda-tracer');
```

{{% notice note %}}
Because we are importing the AppDynamics tracer into the Lambda functions as a Lambda layer, we do not have to include the tracer as a dependency in the deployed Lambda function code payload.
{{% /notice %}}

Within **handler.js**, there are two different exported functions that act as our two different Lambda functions -- **module.exports.doFunctionAsync** and **module.exports.doFunctionAsync2**. We will be working within the **module.exports.doFunctionAsync** function first. Locate the function. This function will either store information in a DynamoDB instance or call another Lambda function, depending on the API Gateway path the function is called from.

``` javascript
module.exports.doFunctionAsync = async (event, context) => {

    const response = {
        headers: { 'Access-Control-Allow-Origin': '*' }, // CORS requirement
        statusCode: 200,
    };

    if (event.path == "/person/submit") {
        var person = personInfo();
        // TODO: Add in first exit call creation for DynamoDB

        try {


            var person_result = await submitPerson(person);
            var result = {
                status : "PersonCreated",
                data : person
            };

            response.body = JSON.stringify(result);
            response.statusCode = 201;


        } catch (e) {

            response.statusCode = 500;
            var result = {
                status : "Error",
                data : e
            };
            response.body = JSON.stringify(result);


        }
        // TODO: End first exit call

    } else if (event.path == "/person/random") {
        const lambda = new AWS.Lambda();

        var params = {
            FunctionName: context.functionName.replace("lambda-1", "lambda-2"),
            InvocationType: "RequestResponse",
            Payload: '{}'
        };

        try {
            var lambda_resp = await lambda.invoke(params).promise();
            var data = JSON.parse(lambda_resp.Payload);
            var result = {
                status : "Found",
                data : data.Item
            }
            response.body = JSON.stringify(result);
        } catch (e) {
            response.statusCode = 500;
            var result = {
                status : "Error",
                data : e
            };
            response.body = JSON.stringify(result);
        }
    } else {
        var ms = _.random(50, 500);
        await doStuff(ms);

        response.body = JSON.stringify({
            status: 'Hello AppDynamics Lambda Monitoring - Async JS handler from ' + event.path + ". ",
            data: context
        });
    }


    return response;
};
```

In the AppDynamics flowmap, the connection to the DynamoDB instance is represented as a connection to a third-party HTTP endpoint. But by adding some code to the function, we can have the connection represented as a connection to an AWS resource. We will add in the code to create and start the exit call. Below the comment to add in the DynamoDB exit call, add the following code.

``` javascript
        var exitCall = null;
        if (tracer != null) {
            exitCall = tracer.startExitCall({
                exitType: 'CUSTOM',
                exitSubType: 'Amazon Web Services',
                identifyingProperties: {
                    'VENDOR': process.env.CANDIDATE_TABLE + " DynamoDB"
                }
            });
        }
```

This code instantiates a variable to hold the exit call, checks to see if we have a tracer, and starts the exit call if the tracer exists. Finally, let's end the exit call. Locate the comment within the code block to end the first exit call and add the following code:

``` javascript
        if (tracer != null && exitCall != null) {
            tracer.stopExitCall(exitCall);
        }
```

The function should look like the following:

``` javascript
module.exports.doFunctionAsync = async (event, context) => {

    const response = {
        headers: { 'Access-Control-Allow-Origin': '*' }, // CORS requirement
        statusCode: 200,
    };

    if (event.path == "/person/submit") {
        var person = personInfo();
        // TODO: Add in first exit call creation for DynamoDB
        var exitCall = null;
        if (tracer != null) {
            exitCall = tracer.startExitCall({
                exitType: 'CUSTOM',
                exitSubType: 'Amazon Web Services',
                identifyingProperties: {
                    'VENDOR': process.env.CANDIDATE_TABLE + " DynamoDB"
                }
            });
        }
        try {

            var person_result = await submitPerson(person);
            var result = {
                status : "PersonCreated",
                data : person
            };

            response.body = JSON.stringify(result);
            response.statusCode = 201;


        } catch (e) {

            response.statusCode = 500;
            var result = {
                status : "Error",
                data : e
            };
            response.body = JSON.stringify(result);


        }
        // TODO: End first exit call
        if (tracer != null && exitCall != null) {
            tracer.stopExitCall(exitCall);
        }

    } else if (event.path == "/person/random") {
        const lambda = new AWS.Lambda();

        var params = {
            FunctionName: context.functionName.replace("lambda-1", "lambda-2"),
            InvocationType: "RequestResponse",
            Payload: '{}'
        };

        try {
            var lambda_resp = await lambda.invoke(params).promise();
            var data = JSON.parse(lambda_resp.Payload);
            var result = {
                status : "Found",
                data : data.Item
            }
            response.body = JSON.stringify(result);
        } catch (e) {
            response.statusCode = 500;
            var result = {
                status : "Error",
                data : e
            };
            response.body = JSON.stringify(result);
        }
    } else {
        var ms = _.random(50, 500);
        await doStuff(ms);

        response.body = JSON.stringify({
            status: 'Hello AppDynamics Lambda Monitoring - Async JS handler from ' + event.path + ". ",
            data: context
        });
    }


    return response;
};
```

Save **handler.js**. We will now make similar changes to the **module.exports.doFunctionAsync2** function, so locate the function.

``` javascript
module.exports.doFunctionAsync2 = async (event, context) => {

    var id_results, ids, id;

    // TODO: Add second exit call to DynamoDB

    try {
        id_results = await getPersonIds();
    } catch (e) {

        // TODO: End second exit call

        context.fail(e);
    }

    // TODO: End second exit call

    ids = _(id_results.Items).map(function (i) {
        return i.id;
    }).value();



    id = ids[_.random(ids.length - 1)];

    // TODO: Add third exit call to DynamoDB

    try {
        var person = await getPerson(id);

        // TODO: End third exit call

        context.succeed(person);
    } catch (e) {

        // TODO: End third exit call

        context.fail(e);
    }
};
```

{{% notice note %}}
The tracer automatically reports errors that occur within an exit call. If you have a scenario where a resulting execution is considered an error, you can manually report custom exit calls. See [the documentation](https://docs.appdynamics.com/display/PRO45/Python+Serverless+Tracer+API) for more information.
{{% /notice %}}

Since we have already imported the tracer at the top of the file, we do not need to do so again. There are 6 commented sections within this file where we will be adding custom code -- 2 places for starting the exit call and 4 for ending the exit call (2 in a success scenario, 2 in a failure scenario). Locate the to-do comment for adding the second exit call to DynamoDB and add the following code below the comment:

``` javascript
    var exitCall = null;
    if (tracer != null) {
        exitCall = tracer.startExitCall({
            exitType: 'CUSTOM',
            exitSubType: 'Amazon Web Services',
            identifyingProperties: {
                'VENDOR': process.env.CANDIDATE_TABLE + " DynamoDB"
            }
        });
    }
```

Locate the to-do comments for ending the second exit call to DynamoDB (there will be two of them) and add the following code below each comment:

``` javascript
        if (tracer != null && exitCall != null) {
            tracer.stopExitCall(exitCall);
        }
```

We're going to repeat this process for the third exit call. This time, however, we're going to use a different variable name for our exit call. Add the following code below the to-do comment for starting the third exit call to DynamoDB:

``` javascript
    var exitCall2 = null;
    if (tracer != null) {
        exitCall2 = tracer.startExitCall({
            exitType: 'CUSTOM',
            exitSubType: 'Amazon Web Services',
            identifyingProperties: {
                'VENDOR': process.env.CANDIDATE_TABLE + " DynamoDB"
            }
        });
    }
```

Finally, locate the to-do comments for ending the third exit call to DynamoDB (there will be two of them) and add the following code below each comment:

``` javascript
        if (tracer != null && exitCall2 != null) {
            tracer.stopExitCall(exitCall2);
        }
```

{{% notice note %}}
Information on the variables to pass when starting the exit call can be fouond in the [product documentation for the NodeJS tracer](https://docs.appdynamics.com/display/PRO45/Customize+the+Node.js+Serverless+Tracer).
{{% /notice %}}

The code for the **module.exports.doFunctionAsync2** function should look like the following:

``` javascript
module.exports.doFunctionAsync2 = async (event, context) => {

    var id_results, ids, id;

    // TODO: Add second exit call to DynamoDB
    var exitCall = null;
    if (tracer != null) {
        exitCall = tracer.startExitCall({
            exitType: 'CUSTOM',
            exitSubType: 'Amazon Web Services',
            identifyingProperties: {
                'VENDOR': process.env.CANDIDATE_TABLE + " DynamoDB"
            }
        });
    }
    try {
        id_results = await getPersonIds();
    } catch (e) {

        // TODO: End second exit call
        if (tracer != null && exitCall != null) {
            tracer.stopExitCall(exitCall);
        }

        context.fail(e);
    }

    // TODO: End second exit call
    if (tracer != null && exitCall != null) {
        tracer.stopExitCall(exitCall);
    }


    ids = _(id_results.Items).map(function (i) {
        return i.id;
    }).value();



    id = ids[_.random(ids.length - 1)];

    // TODO: Add third exit call to DynamoDB
    var exitCall2 = null;
    if (tracer != null) {
        exitCall2 = tracer.startExitCall({
            exitType: 'CUSTOM',
            exitSubType: 'Amazon Web Services',
            identifyingProperties: {
                'VENDOR': process.env.CANDIDATE_TABLE + " DynamoDB"
            }
        });
    }
    try {
        var person = await getPerson(id);

        // TODO: End third exit call
        if (tracer != null && exitCall2 != null) {
            tracer.stopExitCall(exitCall2);
        }

        context.succeed(person);
    } catch (e) {

        // TODO: End third exit call
        if (tracer != null && exitCall2 != null) {
            tracer.stopExitCall(exitCall2);
        }

        context.fail(e);
    }
};
```

The final contents for **handler.js** can be found [here](https://github.com/Appdynamics/appd_aws_lambda_lab/blob/instrumented/node/handler.js).

Save **handler.js**. We have added custom exit calls to our Lambda functions, and now we will deploy our updates. Switch back to the terminal window in the Cloud9 workspace. Change to the *node* directory under the repository, then execute a deploy of the NodeJS Lambda function as shown in the snippet below.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/node
serverless deploy -r $AWS_REGION
```

Allow about 5 minutes for the updated functions to be deployed and start being executed. Then switch back to the window where AppDynamics is running and click on **Application Dashboard** in the application menu on the left-hand side of the screen. You will now see the two NodeJS Lambda functions connected to an AWS resource on the flowmap (in this example it is our DynamoDB instance).

![image](/images/instrumenting_lambda_functions/node/Node_DynamoDB_Exit_Call.png)

{{% notice note %}}
The custom exit call to the DynamoDB instance that we just added and the HTTP endpoint that was auto-detected are the same call -- we're just representing it with a different icon for ease of understanding.
{{% /notice %}}

You did it!! Not only did you configure NodeJS functions for instrumentation by AppDynamics, but you also added in code to create custom exit calls to AWS resouorces! By using AppDynamics for monitoring our Lambda functions, you've given the observability and monitoring teams full visibility into more serverless aspects of the application. Well done!

![image](https://media.giphy.com/media/QEYYlJqOaEhXrjTrOH/giphy.gif)

If you have not done so already, it's time to move on to instrumenting [Python Lambda functions]({{< ref "../51_Python/" >}}) or [Java Lambda functions]({{< ref "../53_Java/" >}}). If this is the end of your journey, let's [clean up]({{< ref "../../60_Teardown/" >}}) any resources we may have used.
