+++
title = "Automatic Instrumentation"
chapter = false
weight = 2
+++

## Add Custom Code for Automatic Instrumentation

Monitoring for AWS Lambda functions written in Java is accomplished by using a tracer SDK. To implement this, we will be leveraging a combination of deployment configuration and adding custom code to our Lambda functions. We have taken care of the deployment configuration in the previous section, and now it's time to add the custom code to handle the instrumentation.

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

### Update the Class / Method Declaration

Next, expand the *src* folder, then expand the following folders: *main*, *java*, *com*, *appdynamics*, *lambda*. The next file we will be updating is **BackEndHandler.java**, so go ahead and open that file. We will also be updating **FrontEndHandler.java** in the next module, so go ahead and open that file as well.

![image](/images/instrumenting_lambda_functions/java/Java_Lambda_Files.png)

AWS Lambda functions in Java require code changes to handle instrumentation. When writing Lambda functions in Java, they will inherit from one of two interfaces -- *RequestHandler* or *RequestStreamHandler*. If the function class inherits from *RequestStreamHandler*, then we can take advantage of more automated instrumentation provided by the tracer SDK.

{{% notice note %}}
Automatic tracer instrumentation only works with functions that inherit from *RequestStreamHandler*. It uses default configurations to manage the transaction, handle correlation, and report errors. See [the documentation](https://docs.appdynamics.com/latest/en/application-monitoring/install-app-server-agents/serverless-apm-for-aws-lambda/java-serverless-tracer) for more information on when automatic instrumentation can be leveraged.
{{% /notice %}}

The first thing we need to do is to import the appropriate tracer resources into the class. Locate the comment labeled `//TODO: import AppDynamics tracer classes` and add the following import statements:

``` java
import com.appdynamics.serverless.tracers.aws.api.AppDynamics;
import com.appdynamics.serverless.tracers.aws.api.MonitoredRequestStreamHandler;
import com.appdynamics.serverless.tracers.aws.api.ExitCall;
import com.appdynamics.serverless.tracers.aws.api.Tracer;
import com.appdynamics.serverless.tracers.aws.api.Transaction;
```

The beginning section of **BackEndHandler.java** should now look like this:

``` java
package com.appdynamics.lambda;

//TODO: import AppDynamics tracer classes
import com.appdynamics.serverless.tracers.aws.api.AppDynamics;
import com.appdynamics.serverless.tracers.aws.api.MonitoredRequestStreamHandler;
import com.appdynamics.serverless.tracers.aws.api.ExitCall;
import com.appdynamics.serverless.tracers.aws.api.Tracer;
import com.appdynamics.serverless.tracers.aws.api.Transaction;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.charset.Charset;
import java.util.List;
import java.util.Map;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestStreamHandler;
import com.amazonaws.util.IOUtils;
import com.appdynamics.lambda.dal.CommerceOrder;
import com.fasterxml.jackson.core.JsonParseException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
```

To complete the automatic instrumentation, we will be making changes to two different parts of **BackEndHandler.java**:

- The interface the class inherits from
- The name of the method that is called.

Locate the line where the class name is declared. It should look like this:

``` java
public class BackEndHandler implements RequestStreamHandler {
```

We are going to change the class declaration from implementing *RequestStreamHandler* to extending *MonitoredRequestStreamHandler*. The *MonitoredRequestStreamHandler* class inherits from *RequestStreamHandler* and adds other configurations needed for monitoring. After changing, the class declaration should look like the following:

``` java
public class BackEndHandler extends MonitoredRequestStreamHandler {
```

Finally, we are going to change the name of the method that is called *handleRequest* to be *handleMonitoredRequest*. After changing, the method declaration should look like the following:

``` java
public void handleMonitoredRequest(InputStream input, OutputStream output, Context context) throws IOException {
```

The fully instrumented version of **BackEndHandler.java** can be found [here](https://github.com/Appdynamics/appd_aws_lambda_lab/blob/instrumented/java/src/main/java/com/appdynamics/lambda/BackEndHandler.java).

Save your changes. Now let's verify that our changes will build. Switch back to the terminal window and make sure that you are in the *java* directory. Then issue the command `mvn clean package` to build.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/java
mvn clean package
```

When the build happens, the tracer SDK will be downloaded before the compilation takes place.

Now that we have added instrumentation to one of our Lambda functions, [let's add it to the other Lambda function]({{< ref "./3_Manual_Instrumentation.md" >}}).
