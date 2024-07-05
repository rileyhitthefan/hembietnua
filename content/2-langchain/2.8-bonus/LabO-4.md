---
title: "Lab O-4: Switching models"
date: 2024-07-05T15:36:35+07:00
draft: false
weight: 4
chapter : false
---

Now that you've hopefully had success running some labs, it's time to experiment with replacing one model with another. In order to switch out models, you will generally need to do 2-3 things:

1. Change the model ID. Refer to the [list of model IDs](https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids-arns.html).
2. Change the model inference parameters. Refer to [details on each model's inference parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html) . Each model provider has its own distinct set of parameters that it requires. Inference parameters are not compatible across models, and will throw an exception if you send one provider's parameters to another provider's model.
3. Change the prompt. Some prompts might work fine across models. Other model providers, like Mistral, recommend a specific prompt format.

## Copy and modify an existing example
1. Copy code from another lab.

```bash
cp -r ~/environment/workshop/completed/text ~/environment/workshop/labs/model_switch
cd ~/environment/workshop/labs/model_switch
```

2. Open the **text_lib.py** file in the newly created **workshop/labs/model_switch** directory.

3. Replace the original model_id with the target model_id.

   - This will be the contents of the string passed to the `model_id` argument.
   - Set the model id to `mistral.mixtral-8x7b-instruct-v0:1`.

4. Replace the original model's inference parameters with the target models' inference parameters.

   - This will be the object passed to the model_kwargs argument.

```python
{ #Mistral
    "max_tokens" : 512,
    "stop" : [],    
    "temperature": 0,
    "top_p": 0.9,
    "top_k": 50
}
```

5. Replace the original model's prompt format with the target models' prompt format.

   - In this case, we will use Python f-strings to wrap the user-provided prompt with Mistral's recommended "[INST]" tags.

```python
return llm.invoke(f"[INST]{input_content}[/INST]") #return a response to the prompt
```

6. Save the file.

{{%expand "Expand to see the completed code for text_lib.py" %}}
```python
from langchain_community.llms import Bedrock

def get_text_response(input_content): #text-to-text client function

    llm = Bedrock( #create a Bedrock llm client
        model_id="mistral.mixtral-8x7b-instruct-v0:1", #set the foundation model
        model_kwargs={
            "max_tokens" : 512,
            "stop" : [],    
            "temperature": 0,
            "top_p": 0.9,
            "top_k": 50
        }
    )
    
    return llm.invoke(f"[INST]{input_content}[/INST]") #return a response to the prompt
```
{{% /expand%}}


7. Run the streamlit command from the terminal.

```bash
streamlit run text_app.py --server.port 8080
```

8. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

9. Try out some prompts and see the results.

   - `What are you?`
   - `What is a good name for a product that provides large language models?`

10. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully switched models!
{{% /notice %}}