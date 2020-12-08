+++
title = "Manual Instrumentation"
chapter = false
weight = 3
+++

## Add Custom Code for Manual Instrumentation

Monitoring for AWS Lambda functions written in Java is accomplished by using a tracer SDK. To implement this, we will be leveraging a combination of deployment configuration and adding custom code to our Lambda functions. We have taken care of the deployment configuration in the previous section, and now it's time to add the custom code to handle the instrumentation.

If you completed the previous section for Automatic Instrumentation (which you should have), then skip the **Add Tracer Dependency** section and go to the **Add Instrumentation Code** section.

### Add Tracer Dependency

The first thing we need to do is to add the tracer SDK as a dependency to our Java project. In the Cloud9 workspace, expand the *appd_aws_lambda_lab* folder, then expand the *java* subfolder. Next, open up the file **pom.xml** -- this file contains project information (including dependencies used by the Lambda functions, which we will be updating).

![image](/images/instrumenting_lambda_functions/java/Java_pom_XML.png)

We're going to insert the following XML at the beginning of the XML block. This will add the AppDynamics Java Lambda tracer when it is time to compile the Lambda function. Locate the XML comment labeled `<!-- TODO: Add AppDynamics dependency -->` and add the following:

``` xml
    <dependency>
       <groupId>com.appdynamics</groupId>
       <artifactId>lambda-tracer</artifactId>
       <version>20.03.1391</version>
    </dependency>
```

The dependencies XML section should look like the following:

``` xml
  <dependencies>
    <!-- TODO: Add AppDynamics dependency -->
    <dependency>
       <groupId>com.appdynamics</groupId>
       <artifactId>lambda-tracer</artifactId>
       <version>20.03.1391</version>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-lambda-java-log4j2</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-secretsmanager</artifactId>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-lambda</artifactId>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-lambda-java-events</artifactId>
      <version>1.3.0</version>
    </dependency>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-dynamodb</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.8.2</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>2.8.2</version>
    </dependency>
    <dependency>
      <groupId>com.github.javafaker</groupId>
      <artifactId>javafaker</artifactId>
      <version>1.0.2</version>
    </dependency>
  </dependencies>
```

Save your changes and close *pom.xml*.

### Add Instrumentation Code

Next, expand the *src* folder, then expand the following folders: *main*, *java*, *com*, *appdynamics*, *lambda*. The next file we will be updating is **FrontEndHandler.java**.

AWS Lambda functions in Java require code changes to handle instrumentation. When writing Lambda functions in Java, they will inherit from one of two interfaces -- *RequestHandler* or *RequestStreamHandler*. If the function class inherits from *RequestStreamHandler*, then we can take advantage of more automated instrumentation provided by the tracer SDK. In this case, this Lambda function inherits from *RequestHandler*, so we will have to use manual instrumentation.

{{% notice note %}}
Automatic tracer instrumentation only works with functions that inherit from *RequestStreamHandler*. It uses default configurations to manage the transaction, handle correlation, and report errors. See [the documentation](https://docs.appdynamics.com/display/PRO45/Java+Serverless+Tracer) for more information on when automatic instrumentation can be leveraged.
{{% /notice %}}

Throughout this section, it is recommended to save **FrontEndHandler.java** periodically. When we make changes, we are going to approach changes in an atomic manner -- any resources that we open or begin (such as exit calls and transactions) will then be immediately closed or completed in the next step. The first thing we need to do is to import the appropriate tracer resources into the class. Locate the comment labeled `//TODO: import AppDynamics tracer classes` and add the following import statements:

``` java
import com.appdynamics.serverless.tracers.aws.api.AppDynamics;
import com.appdynamics.serverless.tracers.aws.api.Tracer;
import com.appdynamics.serverless.tracers.aws.api.Transaction;
import com.appdynamics.serverless.tracers.dependencies.com.google.gson.Gson;
import com.appdynamics.serverless.tracers.aws.api.ExitCall;
```

The beginning section of **FrontEndHandler.java** should now look like this:

``` java
package com.appdynamics.lambda;

//TODO: import AppDynamics tracer classes
import com.appdynamics.serverless.tracers.aws.api.AppDynamics;
import com.appdynamics.serverless.tracers.aws.api.Tracer;
import com.appdynamics.serverless.tracers.aws.api.Transaction;
import com.appdynamics.serverless.tracers.dependencies.com.google.gson.Gson;
import com.appdynamics.serverless.tracers.aws.api.ExitCall;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ThreadLocalRandom;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

import com.amazonaws.services.lambda.AWSLambda;
import com.amazonaws.services.lambda.AWSLambdaClientBuilder;
import com.amazonaws.services.lambda.model.InvokeRequest;
import com.amazonaws.services.lambda.model.InvokeResult;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.appdynamics.lambda.dal.CommerceOrder;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.github.javafaker.Faker;
```

Next, we will add code for the AppDynamics tracer and transaction as well as an exit call to the other Lambda function.  Within **FrontEndHandler.java**, locate the comment labeled `TODO: Add variables for the tracer and transaction` and declare the following variables beneath that comment:

``` java
  Tracer tracer = null;
  Transaction txn = null;
  String correlationHeader = "";
```

These variables will contain the AppDynamics tracer, transaction, and correlation header. Next, locate the comment labeled `TODO: Add in code to build tracer.`. Underneath that comment, we're going to add the following block of code:

``` java
  tracer = AppDynamics.getTracer(context);
  
  if (input.containsKey(Tracer.APPDYNAMICS_TRANSACTION_CORRELATION_HEADER_KEY)) {
    correlationHeader = input.get(Tracer.APPDYNAMICS_TRANSACTION_CORRELATION_HEADER_KEY).toString();
  } else {
    ObjectMapper m = new ObjectMapper();
    Map<String, Object> headers = m.convertValue(input.get("headers"),new TypeReference<Map<String, Object>>() {});
    if (headers != null && headers.containsKey(Tracer.APPDYNAMICS_TRANSACTION_CORRELATION_HEADER_KEY)) {
      correlationHeader = headers.get(Tracer.APPDYNAMICS_TRANSACTION_CORRELATION_HEADER_KEY).toString();
    }
  }

  txn = tracer.createTransaction(correlationHeader);
  txn.start();
```

The above block of code accomplishes the following:

- Instantiates the tracer object using default configuration
- Locates the correlation header passed across in the HTTP headers
- Creates and starts the transaction using the located correlation header

{{% notice note %}}
With the above approach, this assume that we are using the default environment variables as specified in the documentation. If you are using different environment variables or storing the AppDynamics information in a vault, you will need to use a config builder to instantiate the tracer. See [the documentation](https://docs.appdynamics.com/display/PRO45/Java+Serverless+Tracer+API) under *Override the Tracer's Defauult Behavior* for how to do this.
{{% /notice %}}

Next, we're going to locate the comment labeled `TODO: Add code to end transaction` and add the following code block beneath that comment to end the transaction prior to returning our response from the Lambda function:

``` java
  if (txn != null) {
    txn.stop();
  }
```

Nice work so far! We've added in the code to instantiate our tracer along with creating, starting, and stopping the AppDynamics transaction. Our last task will be to add an exit call to our other Lambda function. We will be adding code to create and start the exit call, report an error during the exit call if it occurs, and finally end the exit call.

First, let's create and start the exit call. Locate the comment within **FrontEndHandler.java** labeled `TODO: Add exit call`. Once located, add the following code:

``` java
  HashMap<String, String> payload = new HashMap<>();
  ExitCall lambda_exit_call = null;

  if (txn != null) {
    HashMap<String, String> lambda_props = new HashMap<>();
    lambda_props.put("DESTINATION", lambda_to_call);
    lambda_props.put("DESTINATION_TYPE", "LAMBDA");
    lambda_exit_call = txn.createExitCall("CUSTOM", lambda_props);
    String outgoingHeader = lambda_exit_call.getCorrelationHeader();
    lambda_exit_call.start();
    payload.put(Tracer.APPDYNAMICS_TRANSACTION_CORRELATION_HEADER_KEY, outgoingHeader);
  }
```

This code checks to see if we have a valid AppDynamics transaction. If we do, then we do the following:

- Build out hashmap objects of the payload to send to the other Lambda function and the different properties to identify our exit call
- Instantiate the exit call within the context of the transaction
- Retrieve the correlation header for the exit call
- Start the exit call

After this, we need to make one modification to existing code. With our changes above, we now have a payload to send to the other Lambda function so that correlation will take place. Locate the following code within **FrontEndHandler.java**:

``` java
InvokeRequest request = new InvokeRequest().withFunctionName(lambda_to_call).withPayload("{}");
```

We're going to replace the empty payload in the call with the JSON string representation of our payload. The updated line will look like this:

``` java
InvokeRequest request = new InvokeRequest().withFunctionName(lambda_to_call).withPayload(new Gson().toJson(payload));
```

Next, we will add in the code snippet to end the exit call. Locate the comment labeled `TODO: Add code to end exit call` and add the following Java code beneath it to stop the exit call:

``` java
  if (lambda_exit_call != null) {
    lambda_exit_call.stop();
  }
```

Finally, we will add in code that will report an error in the event that the call to the Lambda function does not succeed. Locate the comment labeled `TODO: Add code to report error for exit call` and add the following code beneath the comment:

``` java
if (lambda_exit_call != null) {
  lambda_exit_call.reportError(e);
}
```

The fully instrumented version of **FrontEndHandler.java** can be found [here](https://github.com/Appdynamics/appd_aws_lambda_lab/blob/instrumented/java/src/main/java/com/appdynamics/lambda/FrontEndHandler.java).

Save your changes. Now let's verify that our changes will build. Switch back to the terminal window and make sure that you are in the *java* directory. Then issue the command `mvn clean package` to build.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/java
mvn clean package
```

Almost there! Now we will redeploy the updated Java Lambda functions. Make sure that you are in the *java* directory and issue the following command in the terminal window:

``` bash
serverless deploy -r $AWS_REGION
```

After the deploy completes, wait about 10 minutes for the previous versions of the Java Lambda functions to stop executing and the new versions to start executing. Switch back to the browser tab where AppDynamics is running and make sure you are in your application. Click on the *Tiers & Nodes* button on the left-hand side of the AppDynamics browser window. You should see the Java Lambda functions appearing there.

![image](/images/instrumenting_lambda_functions/java/Java_Lambda_Tiers.png)

Finally, click on *Application Dashboard* on the left-hand side of the browser window. You should now see the Java Lambda functions appear on the flowmap!

![image](/images/instrumenting_lambda_functions/java/Java_Lambda_Flowmap.png)

You did an amazing job!! You've successfully instrumented your first AWS Lambda functions written in Java using AppDynamics! By adding observability to your Lambda functions using AppDynamics, your IT operations and DevOps teams will be able to monitor how the Lambdas are performing in context of the entire application. Pat yourself on the back -- you deserve it!

![image](https://media.giphy.com/media/cOKjNdJDbqNCm4n0Jm/giphy.gif)

If you have not done so already, it's time to move on to instrumenting [Python Lambda functions]({{< ref "../51_Python/" >}}) or [NodeJS Lambda functions]({{< ref "../52_NodeJS/" >}}). If this is the end of your journey, let's [clean up]({{< ref "../../60_Teardown/" >}}) any resources we may have used.
