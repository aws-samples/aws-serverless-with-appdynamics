+++
title = "Create Cloud9 Workspace"
chapter = false
weight = 1
+++

## Overview

For this exercise, we will be utilizing [AWS Cloud9](https://aws.amazon.com/cloud9/) as the development environment. AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. This section will walk through the steps of provisioning a Cloud9 workspace for use.

{{% notice warning %}}
The Cloud9 workspace should be built by an IAM user with Administrator privileges,
not the root account user. Please ensure you are logged in as an IAM user, not the root
account user.
{{% /notice %}}

<!---
{{% notice info %}}
This workshop was designed to run in the **Oregon (us-west-2)** region. **Please don't
run in any other region.** Future versions of this workshop will expand region availability,
and this message will be removed.
{{% /notice %}}
-->

{{% notice tip %}}
Ad blockers, javascript disablers, and tracking blockers should be disabled for
the cloud9 domain, or connecting to the workspace might be impacted.
Cloud9 requires third-party-cookies. You can whitelist the [specific domains]( https://docs.aws.amazon.com/cloud9/latest/user-guide/troubleshooting.html#troubleshooting-env-loading).
{{% /notice %}}

### Launch Cloud9 in your closest region

To start the provisioning of the Cloud9 workspace, click on a link below that is closest to your location:

- **N. California** [Launch Cloud9 in us-west-1](https://us-west-1.console.aws.amazon.com/cloud9/home?region=us-west-1)
- **Oregon** [Launch Cloud9 in us-west-2](https://us-west-2.console.aws.amazon.com/cloud9/home?region=us-west-2)
- **N. Virginia** [Launch Cloud9 in us-east-1](https://us-east-1.console.aws.amazon.com/cloud9/home?region=us-east-1)
- **Ohio** [Launch Cloud9 in us-east-2](https://us-east-2.console.aws.amazon.com/cloud9/home?region=us-east-2)
- **Ireland** [Launch Cloud9 in eu-west-1](https://eu-west-1.console.aws.amazon.com/cloud9/home?region=eu-west-1)
- **Singapore** [Launch Cloud9 in ap-southeast-1](https://ap-southeast-1.console.aws.amazon.com/cloud9/home?region=ap-southeast-1)

{{% notice info %}}
The minimum environment size for this workshop is **t3.medium** because we will be running part of our workload within the environment.
{{% /notice %}}

### Select your options

Select **Create environment**. Next, name it **AppD-Lambda-Workshop** and click **Next step**.

![image](/images/workshop_setup/Cloud9_Workspace_Name.png)

Choose **Create a new EC2 instance for environment (direct access)** for Environment type. Choose **t3.medium** for Instance type. Select **Amazon Linux 2** for Platform, and leave the other options with the default choices and click **Next Step**.

![image](/images/workshop_setup/Cloud9_Configure_Settings.png)

Finally, verify the selections and click **Create environment**

![image](/images/workshop_setup/Cloud9_Review_Settings.png)

The workspace setup should take approximately 2 minutes. Once the Cloud9 workspace has been setup, you should see a screen similar to the following. There may be a different theme.

![image](/images/workshop_setup/Cloud9_Environment.png)

If a terminal is available in the workspace, then you're ready to start [installing tooling and configuring AppDynamics]({{< ref "2_Install_Tooling_Configure_AppD.md" >}}). If not, open a new terminal by selecting **New Terminal** from the **Window** menu. Then you will be ready to start [installing tooling and configuring AppDynamics]({{< ref "2_Install_Tooling_Configure_AppD.md" >}}).

![image](/images/workshop_setup/Cloud9_New_Terminal.png)