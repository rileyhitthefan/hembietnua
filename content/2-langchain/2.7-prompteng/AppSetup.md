---
title: "Prompt app setup"
date: 2024-07-05T09:55:07+07:00
draft: false
weight: 1
chapter : false
---

## Run the prompt app
1. Select the bash terminal in AWS Cloud9 and paste the following:

```bash
cd ~/environment/workshop/completed/prompt
streamlit run prompt_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

2. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below. You can drag the tab to a different position so that the content will be wider:

![app](/images/2-Bedrock/prompteng/P-1/app.png)

## Prompt tool overview

![app](/images/2-Bedrock/prompteng/P-1/prompt-app.png)

1. Use the **Lab context** radio buttons to determine which text will get inserted into prompts that have a “{context}” placeholder.
2. You can see the context that will be inserted under the **See context** expand panel.
3. Paste prompts from the lab guide into the **Prompt template** text field.
4. Select the model you want to respond to the prompt. When a specific model is indicated in the lab guide, be sure to select that model here.
5. Press the **Run** button to send the prompt to the selected model.
6. The model’s response will show up under **Result**