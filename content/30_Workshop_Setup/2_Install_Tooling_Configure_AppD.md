+++
title = "Install Tooling and Configure AppDynamics"
chapter = false
weight = 2
+++

## Overview

Now with the Cloud9 workspace provisioned and setup, we'll proceed with installing the tooling needed to add observability to our Lambda functions.

The steps we will execute are as follows:

1. Clone the repository for our sandboxed environment.
2. Install necessary tools for deployment and running our sandboxed environment.
3. Install components of sandboxed environment and start application.

### Clone the repository for our sandboxed environment

All of the artifacts that we will use for this exercise are available in the repository for our sandboxed environment. To clone, let's make sure that we are in the environment directory. Execute the following command:

``` bash
pwd
```

If the result is **/home/ec2-user/environment**, then we are ready to go. Otherwise, change to the environment directory by running the following in your terminal window:

``` bash
cd $HOME/environment
```

To clone the repo, run the below command in the terminal:

``` bash
git clone https://github.com/Appdynamics/appd_aws_lambda_lab.git
```

This repo contains the following artifacts:

- Setup and cleanup scripts
- Lambda functions in Java, NodeJS, and Python that will be deployed and instrumented
- A docker-compose application that will be our sandboxed application

![image](/images/workshop_setup/Cloud9_Git_Clone.png)

### Install tooling

Now that we have a local copy of the repository, we will install the tooling needed. All of the tooling installation is contained within a single script. Our next steps are to change to the repository directory and run the installation script.

``` bash
cd appd_aws_lambda_lab
source ./install_tooling.sh
```

This script increases the disk size of the EC2 instance and installs the following components:

- Java 1.8 + Maven
- NodeJS + NPM
- [Serverless framework](https://www.serverless.com/)
- Docker compose

![image](/images/workshop_setup/Cloud9_Install_Tooling.png)

The final outputs of the script will display the versions of each of the components installed.

![image](/images/workshop_setup/Cloud9_Tooling_Versions.png)

### Install Sandbox Environment Components

The sandboxed version of our application runs as a Docker compose application. To install all of the components, first make sure that we are within the repository directory:

``` bash
pwd
```

If the output is **/home/ec2-user/environment/appd_aws_lambda_lab**, then we are ready to go. Otherwise change to the repository directory in the terminal.

``` bash
cd $HOME/environment/appd_aws_lambda_lab
```

Now, execute the setup script by running the following command:

``` bash
source ./setup.sh
```

This script will take anywhere from 5-10 minutes to execute and does the following:

- Saves our current AWS region to an environment variable (which will be used later)
- Creates a login within AppDynamics and the corresponding application
- Deploys our Python, NodeJS, and Java-based Lambda functions to AWS (along with DynamoDB, S3, and API Gateway dependencies)
- Configures our sandbox application to use the recently deployed Lambda functions

All that is left to do is to start our application! We're going to move to the docker-compose folder within our repository and run a startup script.

``` bash
cd $HOME/environment/appd_aws_lambda_lab/docker-compose
source ./start.sh
```

This script pulls down all of the appropriate containers and starts up the docker-compose application.

![image](/images/workshop_setup/Cloud9_Start_Application.png)

{{% notice tip %}}
If you happen to get an error about running out of disk space, then you will need to manually resize the EBS volume attached to the EC2 instance. The setup script resizes the volume to 80 GB. See [the AWS documentation](https://docs.aws.amazon.com/cloud9/latest/user-guide/move-environment.html#move-environment-resize) for how to resize the instance.
{{% /notice %}}

The next thing that we will do is introduce load to our application. By putting our application under load, AppDynamics will be able to not only detect the different components of the application, but it will map out data flows between application components.

Allow 2-3 minutes for all of the application components to fully come online. During this period of time, the following things are happening for each service:

- AppDynamics APM agents start up and register with the controller.
- Application starts and comes online.
- Endpoints become available.

Once the 2-3 minutes have passed, start the load by executing the following command:

``` bash
./startLoad.sh
```

You will see a message stating `appending output to 'nohup.out'`. This is totally expected for how we are running the load generation for the application. We'll let the application run under load for about 5 minutes or so to allow AppDynamics to fully learn the components and data flows between components. Once this amount of time has passed, [we'll view our application in AppDynamics]({{< ref "../40_Logging_in_to_AppDynamics/" >}}) before instrumenting our Lambda functions.
