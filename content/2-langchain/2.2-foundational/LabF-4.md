---
title: "Lab F-4: Intro to Langchain"
date: 2024-06-28T15:18:01+07:00
draft: false
chapter : false
weight: 4
---

In this lab, we will show how to make a basic call to Bedrock using LangChain. While similar in functionality to the previous lab where we called Bedrock through Boto3, in this lab we will use LangChain so you can compare the two approaches. LangChain can abstract away many of the details of using the Boto3 client, especially when you want to focus on text in and text out. If you need the full control enabled by the Boto3 client, it will always be an option for you. Boto3 may require more code, but it will give you full access to the JSON request and response objects.

You can build the application code by copying the code snippets below and pasting into the indicated Python file.

## Create the Python script

1. Navigate to the **workshop/labs/langchain** folder, and open the file **bedrock_langchain.py**

![langchain](/images/2-Bedrock/F-4/1.png)

2. Add the import statements.

   - These statements allow us to use the LangChain library to call Bedrock.
   - You can use the copy button in the box below to automatically copy its code:
```py
from langchain_community.llms import Bedrock
```

3. Initialize the LangChain Bedrock client.

   - This creates a Bedrock client. The client allows us to make standard LangChain calls and have them adapted automatically to work with Bedrock's models.

```py
llm = Bedrock( #create a Bedrock llm client
    model_id="ai21.j2-ultra-v1" #set the foundation model
)
```

4. Set the prompt for the call to Bedrock.

   - With LangChain, the prompt is the minimum payload we need to send to Bedrock. We'll also learn how to set inference parameters in a later lab.

```py
prompt = "What is the largest city in Vermont?"
```

5. Call the Bedrock API.

   - We use LangChains's invoke function to make the call. The Bedrock client also automatically extracts the response text from the response payload, returning only a string.
```py
response_text = llm.invoke(prompt) #return a response to the prompt
```

6. Display the response.

   - This prints the string returned by the LangChain client.

```py
print(response_text)
```

7. Save the file.
Great! Now you are ready to run the script!

## Run the script
1. Select the bash terminal in AWS Cloud9 and change directory.
```bash
cd ~/environment/workshop/labs/langchain
```

2. Run the script from the terminal.
```bash
python bedrock_langchain.py
```

3. The results should be displayed in the terminal.

![langchain](/images/2-Bedrock/F-4/2.png)

{{% notice tip %}}
**Congratulations!**\
You have successfully called Bedrock using LangChain!
{{% /notice %}}
