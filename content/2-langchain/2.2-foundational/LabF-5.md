---
---
title: "Lab F-5: Inference Parameters"
date: 2024-06-28T15:18:04+07:00
draft: false
chapter : false
weight: 5
---

[Inference parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html) are used to configure the response behavior of the foundation model. Inference parameters vary from model to model:

- At a minimum, they can be used to influence the variability (**Temperature, Top P**) and **token length** of the response.
  - We'll learn more about temperature in the [Controlling response variability lab](./LabF-6.md)
  - You can think of **tokens** as either a word or part of a word. The overall ratio of tokens per word varies from model to model. Many common words could be represented as a single token, while less common words might contain multiple tokens.
- **Stop Sequences** are another common parameter, useful for splitting few-shot prompt examples or stopping the model from having a conversation with itself.
You can see the various inference parameters for each model in the Amazon Bedrock console, along with definitions for those parameters.

In this lab, we will customize the inference parameters passed to Amazon Bedrock through the LangChain client. The Bedrock client accepts a `model_kwargs` argument that you can use to set the inference parameters for the model.

{{% notice note%}}
`model_kwargs` objects are not compatible between Bedrock models. if you change the model, you will also need to change the attributes of the object accordingly. The code example in the lab below illustrates the differences between the objects.
{{% /notice %}}

## Create the Python script

1. Navigate to the **workshop/labs/params** folder, and open the file **params.py**

![Params](/images/2-Bedrock/F-5/1.png)

2. Add the import statements.

   - These statements allow us to use the LangChain library to call Bedrock, access command line arguments, and read environment variables.
```py
import sys
from langchain_community.llms import Bedrock
```

3. Create the helper function to get inference parameters based on the model.

   - Each model provider has its own set of inference parameters. This function returns a set of custom parameters based on the first portion of each model's id.
   - You can experiment with different inference parameters using the Text playground in the Bedrock console. You can use the **View API request** button in the playground to get the parameters to use in your code.

```py
def get_inference_parameters(model): #return a default set of parameters based on the model's provider
    bedrock_model_provider = model.split('.')[0] #grab the model provider from the first part of the model id
    
    if (bedrock_model_provider == 'anthropic'): #Anthropic model
        return { #anthropic
            "max_tokens": 512,
            "temperature": 0, 
            "top_k": 250, 
            "top_p": 1, 
            "stop_sequences": ["\n\nHuman:"] 
           }
    
    elif (bedrock_model_provider == 'ai21'): #AI21
        return { #AI21
            "maxTokens": 512, 
            "temperature": 0, 
            "topP": 0.5, 
            "stopSequences": [], 
            "countPenalty": {"scale": 0 }, 
            "presencePenalty": {"scale": 0 }, 
            "frequencyPenalty": {"scale": 0 } 
           }
    
    elif (bedrock_model_provider == 'cohere'): #COHERE
        return {
            "max_tokens": 512,
            "temperature": 0,
            "p": 0.01,
            "k": 0,
            "stop_sequences": [],
            "return_likelihoods": "NONE"
        }
    
    elif (bedrock_model_provider == 'meta'): #META
        return {
            "temperature": 0,
            "top_p": 0.9,
            "max_gen_len": 512
        }
    
    elif (bedrock_model_provider == 'mistral'): #MISTRAL
        return {
            "max_tokens" : 512,
            "stop" : [],    
            "temperature": 0,
            "top_p": 0.9,
            "top_k": 50
        } 

    else: #Amazon
        #For the LangChain Bedrock implementation, these parameters will be added to the 
        #textGenerationConfig item that LangChain creates for us
        return { 
            "maxTokenCount": 512, 
            "stopSequences": [], 
            "temperature": 0, 
            "topP": 0.9 
        }
```

4. Build the function to call Bedrock with the appropriate inference parameters for the model.

   - Here we are instantiating the LangChain Bedrock client, setting the model, and getting the inference parameters for the model.

```py
def get_text_response(model, input_content): #text-to-text client function
    
    model_kwargs = get_inference_parameters(model) #get the default parameters based on the selected model
    
    llm = Bedrock( #create a Bedrock llm client
        model_id=model, #use the requested model
        model_kwargs = model_kwargs
    )
    
    return llm.invoke(input_content) #return a response to the prompt
```

5. Pass the command line parameters to the get_text_response function.

   - We're passing in the first argument (Bedrock Model ID) and second argument (prompt) from the command line.
```py
response = get_text_response(sys.argv[1], sys.argv[2])
```

6. Display the response.
   - This prints the text returned by the model.
```python
print(response)
```

7. Save the file.

## Run the file
1. Select the **bash terminal** in AWS Cloud9 and change directory.
```bash
cd ~/environment/workshop/labs/params
```

2. Run the script from the terminal.
```bash
python params.py "ai21.j2-ultra-v1" "Write a haiku:"
```

The results should be displayed in the terminal:
![Params](/images/2-Bedrock/F-5/2.png)

3. Try using the Mistral model.
```bash
python params.py "mistral.mixtral-8x7b-instruct-v0:1" "[INST]Write a haiku:[/INST]" 
```

4. Try using the Cohere Command model.
```bash
python params.py "cohere.command-text-v14" "Write a haiku:"
```

5. Try using the Meta Llama model.
```bash
python params.py "meta.llama2-70b-chat-v1" "Write a haiku:"
```

6. Try using the Amazon Titan model.
```bash
python params.py "amazon.titan-text-express-v1" "Please write a haiku:"
```

You can learn more about inference parameters in the [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html).

{{% notice tip %}}
**Congratulations!**\
You have successfully called the Bedrock API with inference parameters!
{{% /notice %}}