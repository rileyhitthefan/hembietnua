---
title: "AWS Cloud9 Setup"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 2
---
We will be using [AWS Cloud9](https://aws.amazon.com/cloud9/) as our integrated development environment for this workshop. AWS Cloud9 is one option for building applications with Amazon Bedrock - you can also use your own development tools (VS Code, PyCharm, etc.), [Amazon SageMaker Studio](https://aws.amazon.com/sagemaker/studio/) ,or Jupyter Notebooks.

Below we will configure an AWS Cloud9 [enviroment](https://docs.aws.amazon.com/cloud9/latest/user-guide/environments.html) in order to build and run generative AI applications. An environment is a web-based integrated development environment for editing code and running terminal commands.

{{% notice info %}}
**Assumptions for the following instructions**\
    - AWS Cloud9 will be run from the same account and region where Bedrock foundation models have been enabled.\
    - The acccount and region have a default VPC configured (this is the AWS default).\
\
If you have any challenges below, you may need to access Bedrock from your desktop environment, or create an alternate configuration.
{{% /notice %}}

## Cloud9 Setup instructions

1. In the AWS console, select the region that has Amazon Bedrock foundation models enabled.

2. In the AWS console, search for **Cloud9**.

   - Select **Cloud9** from the search results.

![Cloud9](/images/2-Bedrock/prep/Prep-9.png?classes=border)

3. Select **Create environment**

![Cloud9](/images/2-Bedrock/prep/Prep-10.png)

4. Set the environment details.
   - Set Name to `bedrock-environment`.

![Cloud9](/images/2-Bedrock/prep/Prep-11.png)

5. Set the EC2 instance details.

   - Set **Instance type** to **t3.small**
   - Set **Platform** to **Ubuntu Server 22.04 LTS**
   - Set **Timeout** to **4 hours**

![Cloud9](/images/2-Bedrock/prep/Prep-12.png)

{{% notice warning %}}
**Did you select Ubuntu as your platform?**\
Please double-check that you set Platform to Ubuntu Server 22.04 LTS. This ensures that you will have a Python version that can support LangChain and other critical libraries.
{{% /notice %}}

6. Select the **Create** button.

![Cloud9](/images/2-Bedrock/prep/Prep-13.png)

7. Wait for the environment to be created.

   - You should get a **"Successfully created bedrock-environment"** message in the top banner when ready.
   - In the Environments list, click the **Open** link. This will launch the AWS Cloud9 IDE in a new tab.

{{% notice note %}}
**Handling environment creation errors**\
    - If you get an error message about the selected instance type not being available in the availability zone, delete the environment. Try provisioning again with a different size instance type.
{{% /notice %}}

![Cloud9](/images/2-Bedrock/prep/Prep-14.png)

8. Confirm that the AWS Cloud9 environment loaded properly.
   - You can close the **Welcome** tab
   - You can drag tabs around to the position you want them in.

![Cloud9](/images/2-Bedrock/prep/Prep-15.png)

In this example, the bash terminal tab was dragged up to the top tab strip, and the bottom-aligned panel was closed:

![Cloud9](/images/2-Bedrock/prep/Prep-16.png)

{{% notice tip %}}
**Congratulations!**\
You have successfully configured and launched AWS Cloud9!
{{% /notice %}}
