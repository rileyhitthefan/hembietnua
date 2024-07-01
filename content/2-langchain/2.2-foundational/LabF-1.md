---
title: "Lab F-1: Bedrock Console"
date: 2024-06-28T15:17:27+07:00
draft: false
chapter : false
weight: 1
---
In this lab, we will walk through a few key features of the Bedrock console.

1. Navigate to the **Amazon Bedrock** console.

![bedrock-console](/images/2-Bedrock/F-1/1-bedrock-welcome.png)

2. Expand the **Amazon Bedrock** side menu and select **Examples**.

   - For each example, you can see the prompt, inference configuration, sample response, and API request details.

![bedrock-console](/images/2-Bedrock/F-1/2-examples.png)

3. Choose an example and then select **Open in Playground** to see the example in action.

![bedrock-console](/images/2-Bedrock/F-1/3-playground.png)

4. Select **Run** and review the response.

   - The **Temperature** parameter allows the model to be more "creative" when constructing a response. A temperature of 0 means no randomness - the most likely words are chosen each time. To get more variability in responses, you can set the **Temperature** value higher and run the same request several times.
   - The **Response length** parameter determines the number of tokens to be returned in the response. You can use this to shorten or increase the amount of content returned by the model. If you set the length too low, you might cut off the response before it is completed.
   - You can use the **Info** links to see an explanation of each parameter.

5. Back at the example page, you can view the API Request section to see the JSON payload for the given prompt.

![bedrock-console](/images/2-Bedrock/F-1/4-reqAPI.png)

This is a great way to experiment with different models, prompts, and inference parameters. Spend a few minutes exploring the rest of the console to see its other features.

{{% notice tip %}}
**Congratulations!**\
You have successfully used the Bedrock console to experiment with models!
{{% /notice %}}

