+++
title = "Logging in to AppDynamics"
chapter = false
weight = 40
+++

## Logging in to AppDynamics

Now that our sandboxed application is running under load, we are going to log in to AppDynamics to view the application.

In the terminal window, we're going to change back to the base of our repository, retrieve details of how to login to AppDynamics, and then login to AppDynamics.

``` bash
cd $HOME/environment/appd_aws_lambda_lab
cat workshop-user-details.txt
```

This will list out the information you'll need to login to AppDynamics. In a separate window, navigate to the URL that is next to *Controller URL* in the workshop user details file. Use the data from the workshop user details file to login to AppDynamics.

{{% notice tip %}}
The account and username are two distinct fields that are used as part of logging in.
{{% /notice %}}

![image](/images/instrumenting_lambda_functions/First_Login_AppD.png)

Once logged in, you will see the home screen for AppDynamics. On this screen there should only be one application showing. Click on that application.

![image](/images/instrumenting_lambda_functions/AppD_Home_Screen.png)

Once you click on that application, you will be taken to the home page for that application. On this home page you will see a topological view of all of the application components. In AppDynamics, this view is referred to as a flowmap, since we not only visualize the components of the application but also the data flows between components.

On the flowmap, you'll see the various components of our sandbox environment that are detected and being instrumented. Also within this view, you will see two different gray cloud icons with **HTTP** written inside of them. These icons represent 3rd party HTTP endpoints, and within our application they represent our AWS Lambda functions to which we will be adding instrumentation.

![image](/images/instrumenting_lambda_functions/AppD_First_Flowmap.png)

{{% notice tip %}}
Not seeing all of the components on the flowmap like the picture above? Allow 2-3 additional minutes, then refresh the AppDynamics screen. Repeat until the flowmap fully appears.
{{% /notice %}}

If you click on **Tiers & Nodes** in the left-hand side of this screen, you will see a list of the instrumented components of our application. Note that there are no Lambda functions on this screen -- we will be coming back to this throughout this exercise.

Leave this window open -- we will be switching to and from it throughout this lab. Now that we have visibility into our sandboxed application, it's time to [instrument our Lambda functions]({{< ref "../50_Instrumenting_Lambda_Functions" >}}).
