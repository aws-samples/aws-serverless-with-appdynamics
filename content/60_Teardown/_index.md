+++
title = "Teardown"
chapter = false
weight = 60
+++

## Teardown

Congratulations! You have successfully provided the observability and operations teams with insights into the serverless components of the application! Before we start the teardown process, please complete [this survey](https://www.ciscofeedback.vovici.com/se/6A5348A74CBDCCCF) and let us know how you liked the lab. The feedback from this lab will help us with improving this lab along with future labs.

Now that we have learned how to instrument Lambda functions using AppDynamics, let's free up resources that were created on AWS so that we do not incur any additional costs. Before we do, let's log out of AppDynamics. To log out, click on the gear icon in the top-right corner of the screen then click on the Log out link at the bottom of the pop-up menu.

![image](/images/Logout_AppD.png)

To uninstall all of the components, first make sure that we are within the repository directory:

``` bash
pwd
```

If the output is **/home/ec2-user/environment/appd_aws_lambda_lab**, then we are ready to go. Otherwise change to the repository directory.

``` bash
cd $HOME/environment/appd_aws_lambda_lab
```

Now, execute the setup script by running the following command:

``` bash
source ./teardown.sh
```

This script will take anywhere from 2-3 minutes to execute and does the following:

- Stops our sandbox application (and application load)
- Removes the Lambda functions (along with DynamoDB, S3, and API Gateway dependencies)
- Removes the sandbox application from AppDynamics

Once the teardown script has completed, you are free to delete the Cloud9 instance.
