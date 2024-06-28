---
title: "Amazon Bedrock Setup"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 1
---
We will be using [Amazon Bedrock](https://aws.amazon.com/bedrock/) to access foundation models in this workshop.

Below we will configure model access in Amazon Bedrock in order to build and run generative AI applications. Amazon Bedrock provides a variety of foundation models from several providers.

## Amazon Bedrock Setup instructions
1. Find **Amazon Bedrock** by searching in the AWS console.

![Bedrock](/images/2-Bedrock/prep/Prep-2.png)

2. Expand the side menu.

![Bedrock](/images/2-Bedrock/prep/Prep-3.png)

3. From the side menu, select **Model access**.

![Bedrock](/images/2-Bedrock/prep/Prep-4.png)

4. Select the **Enable specific models button**.

![Bedrock](/images/2-Bedrock/prep/Prep-4'.png)

5. Select the checkboxes listed below to activate the models. If running from your own account, there is no cost to activate the models - you only pay for what you use during the labs. Review the applicable EULAs as needed.

- AI21 > Jurassic-2 Ultra
- Amazon (select Amazon to automatically select all Amazon Titan models)
- Anthropic > Claude 3 Sonnet
- Cohere > Command
- Meta > Llama 2 Chat 70B
- Mistral AI > Mixtral 8x7B Instruct

Select the **Next** button.

![Bedrock](/images/2-Bedrock/prep/Prep-5.png)

6. On the **Review and submit** page, select the **Submit** button.

![Bedrock](/images/2-Bedrock/prep/Prep-6.png)

7. Monitor the model access status. It may take a few minutes for the models to move from **In Progress** to **Access granted** status. You can use the **Refresh** button to periodically check for updates.

8. Verify that the model access status is **Access granted** for the previously selected models.

![Bedrock](/images/2-Bedrock/prep/Prep-8.png)

{{% notice tip %}}
**Congratulations!**\
You have successfully configured Amazon Bedrock!
{{% /notice %}}