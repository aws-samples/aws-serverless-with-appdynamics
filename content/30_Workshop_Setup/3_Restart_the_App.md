+++
title = "Restart the App"
chapter = false
weight = 2
+++

{{% notice note %}}
You only need to follow this section if your Cloud9 instance stops due to inactivity.
{{% /notice %}}

Below are the steps to restart the application.

During the initial setup phase, the setup script created an environment variable to make the configuration and deployment of instrumentation easier. We will need to recreate that environment variable. Execute the following command to save the current region as an environment variable:

``` bash
export AWS_REGION=$(aws configure get region)
```

Next, we will change to the directory where the application resides and start up our application.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/docker-compose
source ./start.sh
```

Allow 5 minutes for services to come fully online, then call the *startLoad.sh* script.

``` bash
./startLoad.sh
```

You're good to go!
