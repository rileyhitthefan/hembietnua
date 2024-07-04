---
title: "Lab S-2: Creating guardrails programmatically"
date: 2024-07-03T16:20:26+07:00
draft: false
weight: 2
chapter : false
---

In this lab, we will programmatically create a guardrail with the same settings as the guardrail created in the AWS Console lab.

You can see the documentation for the create_guardrail method here: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock/client/create_guardrail.html 

We'll go into more details about the various parameters in later labs.

## Create the Python script

1. Navigate to the **workshop/labs/guardrails** folder, and open the file **create_guardrail.py**

![open](/images/2-Bedrock/security/S-2/open.png)

2. Add the import statements.

   - These statements allow us to use the AWS Boto3 library to call Amazon Bedrock.

```python
import boto3, json, random, string
```

3. Add the code to create a guardrail.

   - We're creating a guardrail that filters Bitcoin conversations, toxic content, discussions about the competitor "AnyCompany", and profanity.
   - This guardrail will also mask email addresses and names in the response.

```python
client = boto3.client(service_name='bedrock') #creates a Bedrock client

response = client.create_guardrail(
    name="Guardrail-from-Code-" + "".join(random.choices(string.ascii_lowercase, k=8)),
    description='string',
    topicPolicyConfig={
        'topicsConfig': [
            {
                "name": "Bitcoin",
                "definition": "Providing advice, direction, or examples of how to mine, use, or interact with Bitcoin, including Cryptocurrency-related third-party services.",
                "examples": [
                    "How do I mine Bitcoin?",
                    "What is the current value of BTC?",
                    "Which instance is the best for crypto mining?",
                    "Is mining cryptocurrency against the terms?",
                    "How do I maximize my Bitcoin profits?",
                ],
                "type": "DENY",
            }
        ]
    },
    contentPolicyConfig={
        'filtersConfig': [
            {"type": "SEXUAL", "inputStrength": "HIGH", "outputStrength": "HIGH"},
            {"type": "HATE", "inputStrength": "HIGH", "outputStrength": "HIGH"},
            {"type": "VIOLENCE", "inputStrength": "HIGH", "outputStrength": "HIGH"},
            {"type": "INSULTS", "inputStrength": "HIGH", "outputStrength": "HIGH"},
            {"type": "MISCONDUCT", "inputStrength": "HIGH", "outputStrength": "HIGH"},
            {"type": "PROMPT_ATTACK", "inputStrength": "HIGH", "outputStrength": "NONE"},
        ]
    },
    wordPolicyConfig={
        "wordsConfig": [{"text": "AnyCompany"}],
        "managedWordListsConfig": [{"type": "PROFANITY"}],
    },
    sensitiveInformationPolicyConfig={
        "piiEntitiesConfig": [
            {"type": "NAME", "action": "ANONYMIZE"},
            {"type": "EMAIL", "action": "ANONYMIZE"},
        ],
    },
    blockedInputMessaging="Apologies, this model cannot be used to discuss inappropriate or off-topic content.",
    blockedOutputsMessaging="Apologies, the model's response to your request was blocked.",
)

guardrail_id = response['guardrailId']
```

4. Create a guardrail version.
   - This creates a numbered version of the guardrail that can be used for testing or production purposes.
   - You can also use the built-in “DRAFT” version of a guardrail during development.

```python
response = client.create_guardrail_version(
    guardrailIdentifier=guardrail_id,
)
```

5. Display the guardrail ID.

```python
print(guardrail_id)
```

6. Save the file.

## Run the script
1. Select the bash terminal in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/guardrails
```

2. Run the create guardrail script.
```bash
python create_guardrail.py
```

This should display the newly created guardrail ID.

![app](/images/2-Bedrock/security/S-2/app-in-use.png)

3. (Optional) View the newly created guardrail in the AWS Console.

   - This can be found under Amazon Bedrock > Safeguards > Guardrails.
   - The guardrail will be named something like "Guardrail-from-Code-zzzzzzzz"
   - You may need to refresh the page to see the new guardrail.

![app](/images/2-Bedrock/security/S-2/guardrail-console-list.png)

4. On the guardrail overview page, the details for the guardrail can be found by clicking either **Working Draft** or **Version 1**.

![app](/images/2-Bedrock/security/S-2/guardrail-console-view.png)

{{% notice tip %}}
**Congratulations!**\
You have successfully created a guardrail using the Amazon Bedrock API!
{{% /notice %}}